# power-platform-lessons

A Claude skill with field-tested lessons for Power Apps (canvas and modern controls), Power Fx, Power Automate and Dataverse. Claude applies them before writing formulas, `.pa.yaml`, flow JSON or Dataverse queries. Each lesson is tagged **Verified** (confirmed on Microsoft Learn or tested live) or **Inference** (seen in practice, cause not proven).

## Install

- **Claude.ai / Claude Desktop:** Settings → Capabilities → Skills → Upload skill → `power-platform-lessons.zip`.
- **Claude Code:** copy the `power-platform-lessons/` folder into `~/.claude/skills/`.

## Rebuild the zip

```sh
zip -r power-platform-lessons.zip power-platform-lessons
```
