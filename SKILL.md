---
name: vibecode-submit
description: Draft a vibecode.law project submission from a GitHub repo. Reads the repo, writes a first draft of every form field (title, tagline, about, key features, practice areas, demo URL), captures product screenshots from the live demo or a local dev server, and outputs a ready-to-paste submission folder. Use when the user wants to submit, showcase, or list a project on vibecode.law, or asks to draft a vibecode.law listing from a GitHub link.
license: MIT
user_invocable: true
metadata:
  author: nberk
  version: "1.1.0"
---

# vibecode.law Submission Drafter

Turn a GitHub link into a ready-to-paste vibecode.law submission: every text
field drafted from the repo, real product screenshots captured from the live
demo (or a local dev server), and the few subjective fields collected from the
user. Output is a folder the user copies into the vibecode.law "Share Your
Project" form.

## Usage

```
/vibecode-submit <github-url> [demo-url]
```

- `github-url` (required): e.g. `https://github.com/owner/repo`. If missing, ask for it.
- `demo-url` (optional): the live, interactive demo. If missing, the skill tries
  to discover one, then falls back to running the repo locally.

## Non-negotiables (read first)

1. **Everything is a draft to review, never a final answer.** Present each field,
   let the user edit, never submit on their behalf (the real form is behind login).
2. **Never fabricate screenshots.** Gallery images must be *real captures* of the
   running product. If there is no live demo and the app will not start locally,
   skip the Gallery and tell the user to add images by hand. Do not generate mock
   UI images.
3. **Practice Areas is a closed list.** Only ever output exact strings from the
   list in the Reference section below. If nothing fits, use `Other`.

## Workflow

### Step 1 — Gather inputs
Confirm the `github-url`. Note any `demo-url`. Create an output folder in the
current directory: `vibecode-submission-<owner>-<repo>/` with a `gallery/`
subfolder.

### Step 2 — Read the repo
Shallow-clone into the scratchpad and read it (cloning, not just the API, because
Steps 4–5 need the actual files):

```bash
git clone --depth 1 <github-url> "$SCRATCH/vibecode-src"
```

Then gather signal:
- **README** (the richest source): the project's pitch, features, screenshots, demo links.
- **Metadata** via `gh repo view <owner>/<repo> --json name,description,homepageUrl,url,repositoryTopics,primaryLanguage,licenseInfo,stargazerCount` (supplements topics + homepage; skip if `gh` is unauthenticated).
- **Tech stack** from manifests: `package.json` (deps + `scripts`), `pyproject.toml` / `requirements.txt`, `go.mod`, `Gemfile`, etc.
- **Existing screenshots / assets**: images referenced in the README, and any `logo`/`icon`/`favicon` under `public/`, `assets/`, `static/`, `src/`.

### Step 3 — Draft the text fields
Write a first draft of each, grounded in the repo (do not invent capabilities the
code does not support):

- **Project Title** — the product name, cleaned up (not the repo slug).
- **Tagline** — a short, catchy one-liner. Per house style, offer **5 options** at
  varied lengths and tones and let the user pick. No em-dashes.
- **About the Project** — what it does and what problem it solves (the form's prompt).
- **Key Features** — a short bulleted list of the main features, from the README/code.
- **Practice Areas** — pick the best-fitting subset from the **closed list** in
  Reference. Map by what the tool *does* for legal work (e.g. a contract-review
  tool → `Commercial Contracts` + `Legal Drafting & Document Automation`).
  Validate every pick against the list; never invent a tag. Keep it to the 1–4
  most defensible tags and let the user adjust.
- **Demo URL** — the passed `demo-url`, else `homepageUrl`, else a deployed link
  found in the README (vercel.app / netlify.app / pages.dev / "Live demo"). The
  form wants the *interactive prototype*, not a marketing/landing page; confirm
  with the user.
- **Source Code Availability** — `Available` + the GitHub URL (that's the whole point).

### Step 4 — Capture Gallery screenshots
Target spec: **up to 10 images, min 400×225px (16:9), max 4MB each.** Aim for
4–6 strong shots, not 10 filler ones. Capture at a 16:9 window (1600×900).

**Pick a capture engine:**

- **Preferred — Claude in Chrome extension** (interactive: can click into flows to
  reach screens that need a search or navigation). Load it in one call:
  `ToolSearch select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__resize_window`
  Call `tabs_context_mcp` first. If it reports the extension is **not connected**,
  switch to the fallback rather than asking the user to set it up.
- **Fallback — headless Chrome CLI** (no extension, fully autonomous; captures
  above-the-fold for any directly-navigable URL):
  ```bash
  CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
  "$CHROME" --headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
    --window-size=1600,900 --virtual-time-budget=12000 --screenshot="gallery/01.png" "<url>"
  ```
  `--virtual-time-budget` is essential: it lets JS hydrate and fetch before the
  shot, so you capture real content, not an empty shell. For a screen that needs
  interaction (a specific record/town), navigate **directly** to its URL
  (e.g. `/{state}/{slug}` found in the app's data/index) instead of driving a form.

**Sources, in order:**
1. The passed or discovered **demo URL**.
2. **Run it locally** if there's no demo: detect the dev command from
   `package.json` scripts (`bun run dev` / `npm run dev`) or the framework default,
   start it in the background, detect the port (parse the log, else probe
   3000 / 4321 / 5173 / 8080), capture `http://localhost:<port>`, then stop the
   server. If it binds IPv6-only and the browser can't connect, rebind to `0.0.0.0`.
3. **No web UI (CLI / skill / library):** the product surface is the terminal, so
   capture a clean terminal-style screenshot of the tool running with its real
   command and real output (or a labeled diagram of its flow). Do not fake a web UI.
4. **Nothing runnable:** skip the Gallery and record in the draft that screenshots
   must be added by hand, and why.

**Always verify:** view each captured image and confirm it shows hydrated content,
not a blank frame or an error page. Re-capture (longer `--virtual-time-budget`) if blank.

After capture, normalize with macOS `sips`: ensure each is ≥400×225, convert to
PNG/JPEG, and downscale anything over 4MB. Drop any shot below the minimum.

### Step 5 — Logo (optional)
If Step 2 found a square-ish logo/icon asset, copy it to the output folder and
`sips` it to a square ≥100×100px (≤2MB). Otherwise leave it for the user.

### Step 6 — Collect the human-only fields
These can't be drafted from code. Ask the user:
- **Help Needed** — collaborators, feedback, or specific help? (optional)
- **Video URL** — a YouTube walkthrough, if any. (optional)
- Confirm the **Practice Areas** selection and the **Demo URL**.

### Step 7 — Write the output
Write `vibecode-submission-<owner>-<repo>/draft.md` with one clearly-labeled
section per form field (values ready to copy), a final **Manual steps** checklist
(upload `gallery/` images, set the practice-area chips, paste each field, no
auto-submit), and a note that all copy is an AI first draft to verify. Point the
user at the folder.

## Reference

### Form fields (vibecode.law "Share Your Project")
| Field | Type | Required | Source |
|---|---|---|---|
| Project Title | text | yes | repo |
| Tagline | short text | yes | repo (offer 5) |
| Gallery | ≤10 images, min 400×225px, ≤4MB ea. | yes | captured |
| About the Project | textarea | yes | repo |
| Practice Areas | multi-select (closed list) | yes | repo → closed list |
| Key Features | textarea | yes | repo |
| Logo | square image, min 100×100px, ≤2MB | no | repo asset |
| Video URL | YouTube link | no | human |
| Demo URL | interactive prototype link | no | repo, confirm |
| Help Needed | textarea | no | human |
| Source Code Availability | dropdown | no | `Available` + GitHub URL |

### Practice Areas (closed list — use these exact strings)
Banking & Finance · Bankruptcy & Restructuring · Business Development ·
Commercial Contracts · Competition · Construction · Consumer Protection ·
Corporate · Criminal · Elder Law · Employment · Environmental · Family ·
Healthcare · Immigration · Insurance · Intellectual Property ·
Knowledge Management · Law Firm Operations · Legal Drafting & Document Automation ·
Legal Education & Training · Legal Research & Public Legal Information ·
LegalTech Assessment · Legaltech Tooling · Litigation & Disputes · Medicaid ·
Other · Personal Injury · Planning & Zoning · Privacy & Data Protection ·
Pro Bono & A2J · Real Estate · Regulatory · Tax · Trade ·
White Collar & Investigations · Wills & Estates
