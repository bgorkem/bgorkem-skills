# bgorkem-skills

A personal collection of [Agent Skills](https://www.anthropic.com/news/agent-skills) for [Claude](https://claude.ai). Skills are small, self-contained folders of instructions that teach Claude how to handle specific workflows consistently — without re-explaining context in every conversation.

## Skills in this repository

| Skill | What it does |
| --- | --- |
| [`obsidian-archive`](./skills/obsidian-archive) | Distils the current conversation and saves it as a reference note in an Obsidian vault under `AI/Conversations/`. Requires the [mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) MCP server. |

More skills will be added over time.

## Repository layout

```
.
├── skills/                 # Individual skills, one folder each
│   └── obsidian-archive/
│       └── SKILL.md
├── LICENSE
└── README.md
```

Each skill is a self-contained folder under `skills/` and can be installed independently.

## Installing a skill

### Claude.ai (web / desktop / mobile)

1. Download or clone this repo.
2. Zip the skill folder you want from `skills/` (e.g. `skills/obsidian-archive/` → `obsidian-archive.zip`). The zip's top-level entry must be the skill folder itself, not the `skills/` parent.
3. In Claude.ai, open **Settings → Capabilities → Skills** and upload the zip.
4. Toggle the skill on.

### Claude Code

Copy the skill folder into your Claude Code skills directory:

```bash
git clone https://github.com/bgorkem/bgorkem-skills.git

# User-global (available in every project)
cp -r bgorkem-skills/skills/obsidian-archive ~/.claude/skills/

# Or project-scoped (checked into the project's .claude/skills)
cp -r bgorkem-skills/skills/obsidian-archive /path/to/project/.claude/skills/
```

(Adjust the destination path to match your Claude Code configuration.)

### Dependencies

Each skill documents its dependencies in a `Requirements` section inside its `SKILL.md`. Check that section before installing — some skills require specific MCP servers, plugins, or local tools to function.

## What is a skill?

A skill is a folder containing:

- `SKILL.md` — required file with YAML frontmatter (`name`, `description`, etc.) and Markdown instructions
- Optional `scripts/`, `references/`, `assets/` subfolders for anything the skill bundles

Claude loads the frontmatter into its system prompt so it knows *when* to use the skill; the body loads when the skill triggers. This is called [progressive disclosure](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills).

See Anthropic's [Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) for the full reference.

## Contributing

This is a personal collection, but issues and suggestions are welcome. If you fork and build on these, a link back is appreciated but not required.

## License

[MIT](./LICENSE) — use, modify, and redistribute freely.
