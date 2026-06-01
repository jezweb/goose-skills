# goose-skills

Curated [Agent Skills](https://block.github.io/goose/docs/guides/skills) for [Goose](https://block.github.io/goose/) — portable `SKILL.md` know-how a Goose agent reads on demand. Jezweb's working set, vetted before inclusion.

These are **portable** skills (install a tool, a reusable procedure) — not project-specific ones. Project skills live with their project (e.g. `setup-office-town` lives in [office-town-cloud](https://github.com/jezweb/office-town-cloud)).

## The quality bar

A skill being *available* (here, on skills.sh, agentskills, anywhere) is not a reason to use it. Every skill in this repo has earned its place against:

| Check | Why |
|---|---|
| **Sourced, not remembered** | Install steps / facts come from the live tool or official docs, with the version noted — never from model memory. A wrong version or checksum baked into a skill misleads every future session silently. |
| **Verifies its own work** | Ends with an end-to-end check that *proves* the thing works (not "it should now work"). |
| **Safe by default** | No blind `curl \| bash`. Downloads are checksum-verified where possible. Irreversible / account / money / credential steps are handed to the human, not done by the agent. |
| **Goal + failure mode, not a brittle recipe** | Describes what "done" looks like and what to watch for, so it ages well as tools change. |
| **Earns its tokens** | Removing any line would leave an agent unable to act correctly. Tables over prose. |

If a candidate skill fails these, we rewrite it to pass or leave it out.

## Skills

| Skill | Category | What it does |
|---|---|---|
| [`install-officecli`](skills/install-officecli/SKILL.md) | setup | Install the OfficeCLI binary (create/read/edit `.docx`/`.xlsx`/`.pptx`, no Microsoft Office) + register it as a Goose MCP server. Checksum-verified. |
| [`install-gws`](skills/install-gws/SKILL.md) | setup | Install the official Google Workspace CLI (`gws`) for Gmail/Drive/Docs/Sheets/Calendar from the shell. |

## Installing a skill into Goose

Goose discovers skills in `~/.config/goose/skills/` (and a project's `.goose/skills/`). To add one from this repo:

```bash
# one skill
mkdir -p ~/.config/goose/skills
cp -r skills/install-officecli ~/.config/goose/skills/

# or the whole set
cp -r skills/* ~/.config/goose/skills/
```

The agent then reads the relevant `SKILL.md` when the task matches its `description`. (Goose loads the front-matter `description` to decide relevance — keep it specific.)

## Contributing a skill

1. One folder per skill: `skills/<name>/SKILL.md` (+ any helper files).
2. Front matter: `name`, `version`, `description` (specific — it's the trigger), optional `metadata`.
3. Run it against the quality bar above. If it installs a binary, pin + checksum the version and note where you sourced it.
4. Add a row to the Skills table.

## License

Skills are MIT. The tools they install carry their own licenses (noted per skill).
