# vibecode-submit

An agent skill, written as plain markdown so **[Claude Code](https://claude.com/claude-code),
Codex, Cursor, or any other coding agent** can run it, that turns a GitHub link
into a ready-to-paste [vibecode.law](https://vibecode.law) project submission.

Point it at a repo and it drafts every field on the "Share Your Project" form,
captures real product screenshots, and hands you a folder you copy straight into
the form. You review and tweak; it never submits for you.

## Install

There's nothing to install in the usual sense — `SKILL.md` is the entire
workflow, written as plain instructions any coding agent can read and follow.

1. Clone this repo, or download the ZIP from the GitHub page (**Code → Download ZIP**):
   ```bash
   git clone https://github.com/nberk/vibecode-submit
   ```
2. Tell your coding agent to follow it:
   > Follow the instructions in `SKILL.md` to draft a vibecode.law submission
   > for `<github-url>`.

That's the whole install for Codex, Cursor, or any other agent. **Claude Code**
users have one extra option: dropping the folder into `~/.claude/skills/`
registers it as a native skill, so `/vibecode-submit <github-url>` works as a
shortcut for the same instructions —

```bash
git clone https://github.com/nberk/vibecode-submit ~/.claude/skills/vibecode-submit
```

— then verify with `/vibecode-submit` in Claude Code. Purely a convenience;
the workflow followed is identical either way.

## Use

Tell your agent to follow `SKILL.md`, passing a GitHub URL (and, optionally, a
demo URL):

> Follow SKILL.md to draft a vibecode.law submission for `https://github.com/owner/repo`.

Claude Code users can shortcut this with the slash command:
```
/vibecode-submit https://github.com/owner/repo
/vibecode-submit https://github.com/owner/repo https://your-live-demo.com
```

If you don't pass a demo URL, the skill tries to find one, and falls back to
running the project locally to capture screenshots.

## What you get

A folder named `vibecode-submission-<owner>-<repo>/` containing:

- **`draft.md`** — every form field filled in, ready to copy: title, tagline
  (5 options to choose from), about, key features, practice-area tags, demo and
  source links.
- **`gallery/`** — captured product screenshots, sized to the form's spec.
- a **logo** image if one was found in the repo.

## How it works

The three hard parts (read a repo, write the marketing copy, capture
screenshots) are all things most coding agents can already do, so a plain
markdown workflow is the least code and needs no servers, API keys, or
agent-specific plumbing. Two principles shape it:

- **Screenshots are captured, never generated.** Real submissions need real
  images, so it captures from your live demo or a local dev server, and abstains
  rather than fabricate a fake UI.
- **Practice Areas is a fixed list.** The form offers ~37 set tags; the skill
  maps your project to the exact tags the form accepts, never invented ones.

Fields that code can't know (Help Needed, Video URL) are asked, not guessed.

## Requirements

- A coding agent that can read files, clone a repo, and run shell commands —
  Claude Code, Codex, Cursor, or similar.
- `git` (to read the repo)
- For screenshots: Google Chrome. It's used headlessly (no setup needed) on any
  agent; on Claude Code with the Claude in Chrome extension connected, that's
  used instead for interactive capture.

## Keeping it current

If vibecode.law changes its form, update the field table and the Practice Areas
list in `SKILL.md` (the Reference section). Those two blocks are what the skill
drafts against.

## License

MIT. See [LICENSE](LICENSE).
