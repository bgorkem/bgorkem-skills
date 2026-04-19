---
name: obsidian-archive
description: Summarise the current conversation and save it as a reference note in the user's Obsidian vault under AI/Conversations/. Use this skill whenever the user asks to archive, save, log, remember, capture, or file away the conversation (or "this chat", "what we learned", "this session") into Obsidian. Also trigger on phrases like "save this to my vault", "add this to Obsidian", "write this up as a note", or "archive this conversation" — even if Obsidian isn't explicitly named, if the context is about persisting conversation takeaways for later reference, use this skill. Do NOT use for saving arbitrary content the user pastes in; this is specifically for distilling the live conversation.
compatibility: Requires the mcp-obsidian MCP server (MarkusPfundstein/mcp-obsidian) connected to an Obsidian vault with the Local REST API community plugin enabled. Obsidian must be running. Tools used: obsidian_list_files_in_dir, obsidian_append_content, obsidian_get_file_contents.
license: MIT
metadata:
  author: Bulent Gorkem
  version: 1.1.0
  mcp-server: mcp-obsidian
---

# Obsidian Archive

Distil the current conversation into a concise reference note and save it to the user's Obsidian vault at `AI/Conversations/`, one file per conversation.

The goal is a **future-user-friendly** note: something they can grep or backlink to months later and immediately recall what was figured out. Not a transcript. Not a stenographer's record. A distillation.

## When to use

Trigger on any of:

- "Archive this conversation", "save this chat", "add this to Obsidian", "log this"
- "Save what we learned", "capture this", "write this up", "file this away"
- "Remember this for later", "turn this into a note"

If the user's intent is ambiguous (e.g. "save this" could mean saving a file they uploaded), ask one quick clarifying question before proceeding.

## Dependencies

This skill orchestrates tools from the `mcp-obsidian` MCP server. If these aren't loaded into context, call `tool_search(query="obsidian")` first to load them.

| Tool | Purpose |
| --- | --- |
| `obsidian_list_files_in_dir` | Check for filename collisions in `AI/Conversations/` |
| `obsidian_append_content` | Create the note (creates the file and parent folders if they don't exist) |
| `obsidian_get_file_contents` | Read existing archive when the user wants to update it |

If the MCP server isn't connected, stop and tell the user — don't fall back to a different vault location or save strategy.

## Workflow

### Step 1: Decide the filename

Format: `YYYY-MM-DD - <slug>.md`

- Use the **current date** (today, not the conversation's start date unless that's obviously different).
- The slug is a 3–6 word kebab-case summary of the conversation's main subject. Examples:
  - `2026-04-19 - obsidian-cli-move-files.md`
  - `2026-04-19 - fxblotter-websocket-architecture.md`
  - `2026-04-19 - nestjs-bullmq-rate-limiter.md`
- Avoid generic slugs like `chat`, `conversation`, `discussion`.

Before writing, call `obsidian_list_files_in_dir(dirpath="AI/Conversations")` to check for collisions. If the exact filename exists, append `-2`, `-3`, etc.

If the folder doesn't exist yet, `obsidian_append_content` will create it — no need to create it separately.

### Step 2: Build the note content

Use this exact structure. Sections may be omitted **only** if genuinely empty (don't pad with filler).

````markdown
---
date: YYYY-MM-DD
tags: [conversation, <2-4 topical tags>]
---

# <Descriptive title — same subject as the slug, but human-readable>

> One-to-two sentence TL;DR of what this conversation was about and what came out of it.

## Key decisions & conclusions

- <Bullet. What was decided or concluded. Terse. Active voice.>
- <Another. Include the *why* when non-obvious.>

## Code & commands that worked

```<language>
<only commands/snippets that were confirmed working — not drafts that failed>
```

Add brief context above each block if needed (one line).

## Follow-ups & open questions

- [ ] <Thing explicitly deferred, or a natural next step>
- [ ] <Open question that wasn't resolved>

## Related

- [[<Existing vault note if relevant>]]
- <External URL with short description, if a key reference>
````

**Content rules — important:**

- **Distil, don't transcribe.** A 40-turn conversation might compress to 15 lines. That's fine. That's the point.
- **Include what failed only if the failure is instructive.** "We tried X, it didn't work because Y, so we switched to Z" is worth capturing. Dead ends that taught nothing are not.
- **Preserve working commands verbatim.** Don't paraphrase shell commands, API calls, or code. These are the highest-value content.
- **Tag thoughtfully.** 2–4 topical tags in the frontmatter (e.g. `obsidian`, `cli`, `nestjs`, `forex`). Always include `conversation` as one tag. Don't use spaces in tags.
- **Backlink opportunistically.** If the conversation touched a topic likely to have a note in the vault, include a `[[wikilink]]` in Related. Don't invent links to notes that don't exist.
- **No "Claude said" / "User asked" framing.** Write in third person / impersonal style, like an engineering lab notebook entry.

### Step 3: Save via MCP

Call `obsidian_append_content` with:

- `filepath`: `AI/Conversations/YYYY-MM-DD - <slug>.md`
- `content`: the full markdown from step 2

Since this tool appends (or creates if new), step 1's collision check ensures the file doesn't already exist.

### Step 4: Confirm to the user

Reply with: the path that was saved, and a one-line summary of what's in it. Don't paste the full note back — they can open it in Obsidian.

Example: "Saved to `AI/Conversations/2026-04-19 - obsidian-cli-move-files.md` — covers the `obsidian eval` + `fileManager.renameFile` pattern for moving files with backlink updates."

## Examples

### Example 1: Technical deep-dive with working code

**User says:** "Archive this conversation before I lose it."

**Context:** A 25-turn conversation about moving files in Obsidian via CLI, culminating in a working two-step command.

**Actions:**
1. `obsidian_list_files_in_dir(dirpath="AI/Conversations")` → no collision.
2. Build note with slug `obsidian-cli-move-files`, tags `[conversation, obsidian, cli, mcp]`, TL;DR summarising the CLI-vs-MCP tradeoff, a Key Decisions section listing the `fileManager.renameFile` insight, a Code block with the working two-step command, and a Follow-ups entry to wrap it in a zsh function.
3. `obsidian_append_content(filepath="AI/Conversations/2026-04-19 - obsidian-cli-move-files.md", content=<markdown>)`.

**Result:** "Saved to `AI/Conversations/2026-04-19 - obsidian-cli-move-files.md` — covers the CLI move pattern with backlink updates."

### Example 2: Too-short conversation

**User says:** "Save this chat to Obsidian."

**Context:** Three turns about the weather.

**Action:** Don't archive. Reply: "This conversation's pretty short — it covered today's forecast and nothing that'd benefit from a reference note. Want to archive it anyway, or skip?"

### Example 3: Update an existing archive

**User says:** "Add today's findings to yesterday's conversation note."

**Actions:**
1. `obsidian_list_files_in_dir(dirpath="AI/Conversations")` to find the most recent matching file.
2. `obsidian_get_file_contents` to confirm it exists and read context.
3. `obsidian_append_content` with a new `## Update 2026-04-19` section containing today's additions — don't rewrite the existing content.

## Edge cases

- **Multiple distinct topics.** If the conversation genuinely covered unrelated subjects, offer to save two separate notes rather than one Frankenstein file. One file per conversation is the default, but topic coherence matters more.
- **User asks for a different location/format.** Respect the override for that call, but don't treat it as a permanent change. If they want the default changed, they should update the skill itself.
- **Vault has a different folder structure.** This skill assumes `AI/Conversations/`. If that path doesn't fit the user's vault, ask once whether to use an alternative, don't guess.

## Troubleshooting

**Error: `404 Not Found` from obsidian_append_content**
Cause: Obsidian isn't running, or the Local REST API plugin is disabled/misconfigured.
Solution: Ask the user to confirm Obsidian is open and the Local REST API plugin is enabled. Don't retry blindly.

**Tool call fails with "tool not found"**
Cause: The mcp-obsidian tools aren't loaded in this session.
Solution: Call `tool_search(query="obsidian")` first, then retry.

**Filename collision after appending `-2`, `-3`, `-4`**
Cause: The user is archiving many conversations on the same subject on the same day.
Solution: Ask whether they want to update an existing file (see Example 3) rather than keep creating new ones.

**User's vault uses a different conversations folder**
Cause: Vault structure differs from the default assumption.
Solution: Ask which folder they'd prefer for this archive. Don't silently save to `AI/Conversations/` if they've indicated otherwise.

## What NOT to do

- Don't save the raw conversation transcript. Ever.
- Don't include timestamps for individual turns — only the date in frontmatter.
- Don't speculate about the user's unstated intent when writing Key Decisions. If they didn't actually decide X, don't claim they did.
- Don't use emoji in the note body unless the user does. The vault is an engineering reference, not a scrapbook.
- Don't add a note you can't justify the existence of. Quality over quantity.
