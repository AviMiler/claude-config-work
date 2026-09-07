---
name: hebrew-office-docs
description: >
  Creating Hebrew (or any RTL-language) Office documents — Word (.docx),
  PowerPoint (.pptx), Excel (.xlsx). Use whenever a docx/pptx/xlsx file will
  contain Hebrew or other RTL text, on top of the generic `docx`/`pptx`/`xlsx`
  skills that handle file mechanics. Covers the mandatory per-run `lang`
  attribute that prevents Word/PowerPoint from silently breaking bidi word
  ordering, plus RTL paragraph/sheet direction settings.
---

# Hebrew / RTL Office Documents

⚠️ Read First — this skill is an addition on top of the base `docx`,
`pptx`, and `xlsx` skills, not a replacement. Load the matching base skill
for file mechanics (creating/editing the file, tables, images, formulas,
etc.); load this skill in addition whenever the content includes Hebrew or
other RTL text.

## The Core Rule (do not skip)

**Every run of Hebrew (or other RTL) text in a docx/pptx file must have its
language explicitly set to `lang="he-IL"` (or the correct RTL locale) on
the run itself.**

The generator's default run language is `lang="en-US"`. Even when
`rtl`/`bidi`/`rtlMode` is set correctly for the paragraph or text box, a run
still tagged `en-US` makes Word/PowerPoint apply LTR bidi word-ordering
rules to it — the text renders with words or punctuation in the wrong
order, silently and without any visible error. Setting `rtl` alone is not
enough. This is easy to miss because the document *looks* fine in a quick
glance and only breaks on closer reading or specific word orders.

**Mixed Hebrew/Latin text must be split into separate runs by language
before writing.** A single run cannot carry two `lang` values — Hebrew
segments and Latin/numeric segments (e.g. an English product name or a
number inside a Hebrew sentence) each need their own run, each tagged with
its own correct language.

## docx (python-docx)

```python
from docx.oxml.ns import qn

def set_run_language(run, lang="he-IL"):
    rPr = run._element.get_or_add_rPr()
    lang_el = rPr.find(qn('w:lang'))
    if lang_el is None:
        lang_el = rPr.makeelement(qn('w:lang'), {})
        rPr.append(lang_el)
    lang_el.set(qn('w:val'), lang)
    lang_el.set(qn('w:bidi'), lang)  # bidi lang is what governs RTL runs
```

Apply this to **every run** containing Hebrew text, immediately after
creating it — not just once at the paragraph level. Also set the
paragraph's `bidi` property (`paragraph.paragraph_format` /
`w:bidi` on `w:pPr`) so paragraph-level layout (alignment, indentation
direction) is RTL; that alone does not fix run-level word ordering, hence
the rule above.

## pptx (python-pptx)

Same principle, per run in each text frame:

```python
from pptx.oxml.ns import qn

def set_run_language(run, lang="he-IL"):
    rPr = run._r.get_or_add_rPr()
    rPr.set('lang', lang)
```

Set the paragraph/text-frame RTL flag too (`p.paragraph_format` /
`a:pPr rtl="1"`), but it does not substitute for the per-run `lang`.

## xlsx (openpyxl)

Excel does not carry a per-run language tag the way Word/PowerPoint do, but
still needs explicit RTL handling or Hebrew text and layout will look
wrong:

- Set sheet direction: `worksheet.sheet_view.rightToLeft = True` for
  sheets that are primarily Hebrew.
- Set cell alignment explicitly RTL where mixed-direction content would
  otherwise be misread: `cell.alignment = Alignment(readingOrder=2)`
  (2 = RTL).

## Splitting mixed-language text into runs

Before writing any paragraph/text-box content that mixes Hebrew and
Latin/numeric text, split it into segments by script/language first (e.g.
regex on Hebrew Unicode block `֐-׿` vs. everything else), write
each segment as its own run, and tag each run's `lang` accordingly
(`he-IL` for Hebrew segments, `en-US` — or leave default — for Latin/
numeric segments). Do not write a single run spanning both.

## Verification

After generating the file, unzip it and grep the run XML
(`word/document.xml`, `ppt/slides/slideN.xml`) for `w:lang`/`lang=` to
confirm every Hebrew run carries `he-IL` (or `bidi="he-IL"`), not the
default `en-US`. A hook reminds you of this rule whenever the `docx`,
`pptx`, or `xlsx` skill is invoked — but the hook cannot verify the actual
XML, so this manual check is still required before considering the file
done.
