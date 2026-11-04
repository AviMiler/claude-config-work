---
name: code-standards
description: >
  General code-writing rules for every stack, on top of agentic-code-local.
  Load during BUILD of every code change.
---

# Code Standards

Load together with `agentic-code-local`. Topic skills (for example `chrome-extension-dev`) add stack-specific rules; on conflict, the topic skill wins. If still unclear, ask the user.

## Rules

### 1. File and folder organization
- One file, one responsibility. If a file does two different jobs, split it.
- One folder per topic (for example `auth/`, `billing/`), not one folder per technical layer. Split by role (`handlers`, `models`) only inside a topic that has grown large.
- Follow the language's usual convention: Python packages per domain, JS/TS feature folders, Go `internal/`, Java package per domain. If unsure, use what the framework's own generator creates.
- Apply this to files you create. If an existing file already breaks this rule, do not split it and do not mention it. Do only what the request asks.
