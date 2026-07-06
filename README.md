# vibecode-submit

An agent skill, built first for [Claude Code](https://claude.com/claude-code)
and written as plain markdown so **Codex, Cursor, or any other coding agent**
can run it too, that turns a GitHub link into a ready-to-paste
[vibecode.law](https://vibecode.law) project submission.

Point it at a repo and it drafts every field on the "Share Your Project" form,
captures real product screenshots, and hands you a folder you copy straight into
the form. You review and tweak; it never submits for you.

## Install

**With Claude Code:** add this skill to your personal skills folder. Pick
whichever is easier:

**Option A — clone it (if you use git):**

```bash
git clone https://github.com/nberk/vibecode-submit ~/.claude/skills/vibecode-submit
```

**Option B — download the ZIP (no git needed):**

1. On the GitHub page, click **Code → Download ZIP**.
2. Unzip it. You'll get a folder named `vibecode-submit` (or `vibecode-submit-main`; rename it to `vibecode-submit`).
3. Move that folder into `~/.claude/skills/`. If the folder doesn't exist, create it:
   ```bash
   mkdir -p ~/.claude/skills
   ```

**Verify:** open Claude Code and type `/vibecode-submit`. It should appear in the
list. (You can also just ask Claude: "draft a vibecode.law submission for
`<github-url>`".)

**With Codex, Cursor, or any other coding agent:** no install step. Clone or
open this repo, then tell your agent:

> Follow the instructions in `SKILL.md` to draft a vibecode.law submission for
> `<github-url>`.

`SKILL.md` is self-contained — it doesn't depend on any Claude-Code-specific
feature. The one optional exception (the Claude in Chrome extension in Step 4)
is skipped automatically in favor of a plain headless-Chrome command when it
isn't available.

## Use

**Claude Code:**
```
/vibecode-submit https://github.com/owner/repo
/vibecode-submit https://github.com/owner/repo https://your-live-demo.com
```

**Any other agent:** point it at `SKILL.md` and give it the same two arguments
in plain language.

If you don't pass a demo URL, the skill tries to find one, and falls back to
running the project locally to capture screenshots.

## What you get

A folder named `vibecode-submission-<owner>-<repo>/` containing:

- **`draft.md`** — every form field filled in, ready to copy: title, tagline
  (5 options to choose from), about, key features, practice-area tags, demo and
  source links.
- **`gallery/`** — captured product screenshots, sized to the form's spec.
- a **logo** image if one was found in the repo.

## How it works (and why it's a skill)

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
