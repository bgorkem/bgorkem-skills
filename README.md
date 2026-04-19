# bgorkem-skills

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) hosting a personal collection of [Agent Skills](https://www.anthropic.com/news/agent-skills) for [Claude](https://claude.ai). Skills are small, self-contained folders of instructions that teach Claude how to handle specific workflows consistently — without re-explaining context in every conversation.

## Skills in this repository

| Plugin | What it does |
| --- | --- |
| [`obsidian-archive`](./plugins/obsidian-archive) | Distils the current conversation and saves it as a reference note in an Obsidian vault under a configurable archive folder. Requires the [mcp-obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) MCP server. |

More skills will be added over time.

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json                       # marketplace catalog
├── plugins/
│   └── obsidian-archive/                      # one plugin per skill
│       ├── .claude-plugin/
│       │   └── plugin.json                    # plugin manifest
│       └── skills/
│           └── obsidian-archive/
│               └── SKILL.md                   # the skill itself
├── LICENSE
└── README.md
```

Each plugin is a self-contained folder under `plugins/` and can be installed independently.

## Installing

### Claude Code (recommended)

Add this repo as a marketplace, then install the plugin you want:

```shell
/plugin marketplace add bgorkem/bgorkem-skills
/plugin install obsidian-archive@bgorkem-skills
```

Updates are one command away:

```shell
/plugin marketplace update bgorkem-skills
```

### Claude.ai (web / desktop / mobile)

1. Clone or download this repo.
2. Zip the `SKILL.md` and any sibling files inside a skill folder — the zip's top-level entry must be the skill folder itself. For `obsidian-archive`:
   ```bash
   cd plugins/obsidian-archive/skills
   zip -r ../../../obsidian-archive.zip obsidian-archive
   ```
3. In Claude.ai, open **Settings → Capabilities → Skills** and upload the zip.
4. Toggle the skill on.

### Requirements

Each plugin documents its external dependencies in a `Requirements` section inside its `SKILL.md`. Check that section before installing — some skills require specific MCP servers, plugins, or local tools to function.

## What is a skill?

A skill is a folder containing:

- `SKILL.md` — required file with YAML frontmatter (`name`, `description`) and Markdown instructions
- Optional `scripts/`, `references/`, `assets/` subfolders for anything the skill bundles

Claude loads the frontmatter into its system prompt so it knows *when* to use the skill; the body loads when the skill triggers. This is called [progressive disclosure](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills).

See Anthropic's [skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) and [Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) for the full reference.

## Contributing

This is a personal collection, but issues and suggestions are welcome. If you fork and build on these, a link back is appreciated but not required.

## License

[MIT](./LICENSE) — use, modify, and redistribute freely.
