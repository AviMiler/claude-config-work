D-001 | 2026-11-04 00:00 | done | ask on risky git/rm commands in settings.json | enforce "never commit/push" beyond text
D-002 | 2026-11-04 00:00 | done | on rule-breaking existing files: fix only the request, do not mention splitting | avoid unrequested changes
D-003 | 2026-11-04 00:00 | done | record decisions immediately as received, mark done after DOCUMENT | no loss if session ends early
D-004 | 2026-11-04 00:00 | done | DECISIONS.md lines have ID and status (superseded ignored by CHECK) | file grows, replaced decisions must not conflict
D-005 | 2026-11-04 00:00 | done | fast track for trivial changes | workflow too heavy for small edits
D-006 | 2026-11-04 00:00 | done | remove the branch question | repeated every task
D-007 | 2026-11-04 00:00 | done | agent restrictions stay textual | simple, user test catches deviations
D-008 | 2026-11-04 00:00 | done | no tests -> Conditional Go with manual checks | code reading is not proof
D-009 | 2026-11-04 00:00 | done | sub-agents stay the main way | as before
D-010 | 2026-11-04 00:00 | done | develop to be merged into main at the end | work config replaces personal config
D-011 | 2026-11-04 00:00 | done | remove stale date note, git add not on ask, clean HTML comment | leftovers
D-012 | 2026-11-04 14:26 | done | chrome-extension-dev: full shortening (drop System Permissions, repeated summary, double-injection section; shorten Gotchas; clean style) | too long and off-style for work repo
D-013 | 2026-11-04 14:26 | done | chrome-extension-dev: read the reference file only as needed | save context
D-014 | 2026-11-04 14:26 | done | chrome-extension-dev: remove the 500-line max rule | overlaps with one-responsibility-per-file in code-standards
D-015 | 2026-11-04 14:28 | done | new-project-setup: remove checkout/switch/stash from allow (move to ask); branch stays read-only allow | CLAUDE.md forbids switching/creating branches
D-016 | 2026-11-04 14:28 | done | new-project-setup: keep git push --force in ask (not deny); git-instructions "Never force-push" stays as is | user choice, occasional deliberate use
D-017 | 2026-11-04 14:28 | done | new-project-setup: keep the narrowed per-project permission template | no leak to other projects
D-018 | 2026-11-04 14:28 | done | new-project-setup: delete "Why these choices" and the closing "sane starting point" paragraph | template is self-explanatory
D-019 | 2026-11-04 14:28 | done | move agent files from agents/agents/ to agents/ (update README paths) | maps directly to ~/.claude/agents, same as skills/
D-020 | 2026-11-04 14:30 | done | move the file-organization rule from code-standards into agentic-code-local, delete the code-standards skill, remove all references | one skill instead of two overlapping
D-021 | 2026-11-04 14:30 | done | keep the no-commit/push/PR/merge rule in all current places (no dedupe) | redundancy ensures the agent sees it even when a skill loads alone
D-022 | 2026-11-04 14:39 | done | settings.json: remove defaultMode acceptEdits (allow already covers Edit/Write) | redundant
D-023 | 2026-11-04 14:39 | done | fast track has no DOCUMENT stage (update workflow skill, README, spec-doc-agent flow) | updating AGENT_CONTEXT/CHANGELOG for a typo is too heavy
D-024 | 2026-11-04 14:39 | done | add one line to root README about tracking files (AGENT_CONTEXT, CHANGELOG, PROJECT_MAP) | only defined in skill and agent
D-025 | 2026-11-04 14:39 | done | keep D-001..D-011 timestamps at 00:00 | historical lines, no need to fix
D-026 | 2026-11-04 14:54 | done | agentic-code-local: at every changed place, add a very short comment (what and why, one line) | human reviewer sees each change and its reason in the code
