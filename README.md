# Power Platform Lessons — a Claude skill

Field-tested lessons for building on **Power Apps** (canvas and modern controls), **Power Fx**, **Power Automate** and **Dataverse**, packaged as a [Claude skill](https://support.claude.com/en/articles/12512180-using-skills-in-claude). Claude reads them before it writes a formula, a `.pa.yaml` file, flow JSON or a Dataverse query, so it avoids the mistakes Microsoft Learn doesn't warn you about.

Free to use, copy and change under the [MIT License](LICENSE).

## What it covers

| Section | Examples |
| --- | --- |
| Canvas and modern controls | Which properties each modern control version really has (PA2108/PA2109 on paste), Default vs output properties, galleries and tab lists that keep their selection, component limits |
| Power Fx and delegation | Binding values before a Dataverse filter, what delegates and what doesn't, `IfError` type traps, `ForAll` restrictions, option-set comparisons |
| Studio paste, YAML and tooling | Where a paste lands, `.pa.yaml` rules, parsing `.pa.yaml` in Python, Test vs Live publish slots, what solution exports contain |
| Layout and theming | Auto-layout sizing, theme tokens, clamped font sizes, optional UI conventions |
| Power Automate | Try/catch runs that look green, PowerAppV2 trigger schema, Power Automate Tools (XrmToolBox) JSON, connection references, custom connectors |
| Dataverse, SQL 4 CDS, XrmToolBox | Logical names, `@odata.bind`, TDS endpoint limits, SQL 4 CDS query rules, bulk deletes |
| Working method | One change at a time, diffing exports, when to stop guessing and instrument |

Every lesson is tagged **Verified** (confirmed on Microsoft Learn or tested live) or **Inference** (seen in practice, cause not proven). Claude is told to say so when it relies on an Inference.

## Install

### Claude Code (plugin)

```sh
claude plugin marketplace add MaurizioJJ/power-platform-lessons-skill
claude plugin install power-platform-lessons@power-platform-lessons
```

Or from inside a session: `/plugin install power-platform-lessons --marketplace MaurizioJJ/power-platform-lessons-skill`. Update later with `claude plugin update power-platform-lessons@power-platform-lessons`.

### Claude Desktop or claude.ai (skill upload)

1. Download [`power-platform-lessons.zip`](power-platform-lessons.zip) from this repo.
2. Open **Settings → Capabilities** and make sure **Code execution and file creation** is on (skills need it).
3. In **Skills**, click **Upload skill** and choose the zip.
4. Turn the skill on.

Skills are available on Pro, Max, Team and Enterprise plans. On Team and Enterprise, an admin may need to allow skills first.

### Manual copy (Claude Code)

```sh
git clone https://github.com/MaurizioJJ/power-platform-lessons-skill.git
cp -R power-platform-lessons-skill/skills/power-platform-lessons ~/.claude/skills/
```

## Usage

You don't need to call it. Ask Claude for Power Platform work as usual and it loads the skill when the task fits, for example:

- "Write the Items formula for this gallery, filtered by the signed-in user's region, without delegation warnings."
- "Convert this solution-export flow to Power Automate Tools JSON."
- "Why does my ModernCombobox show empty in view mode?"

To force it, name it: "Use the power-platform-lessons skill."

### Keep your own lessons

The skill looks for a lessons file in your project (for example `docs/power-platform-lessons.md`). If one exists, Claude reads it at the start of a task and adds what it learned at the end, in the same Verified/Inference format. Your project-specific findings then sit next to the general ones.

## Contributing

Issues and pull requests are welcome. A new lesson should be one line, say what to do (not only what went wrong), name the control version when it matters, and carry a **Verified** or **Inference** tag. Please leave out anything that identifies a company, tenant or project.

After editing `skills/power-platform-lessons/SKILL.md`, bump `version` in `.claude-plugin/plugin.json`, then rebuild the zip and validate:

```sh
(cd skills && zip -r ../power-platform-lessons.zip power-platform-lessons -x '*.DS_Store')
claude plugin validate --strict .
```

## Disclaimer

Community notes, not official Microsoft guidance. Power Platform changes often; check behavior in your own environment before relying on a lesson. Not affiliated with or endorsed by Microsoft or Anthropic. Power Apps, Power Automate, Power Fx and Dataverse are trademarks of Microsoft Corporation.

## License

[MIT](LICENSE) — free for personal and commercial use.
