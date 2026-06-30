---
name: vibecode-submit
description: Draft a vibecode.law project submission from a GitHub repo. Reads the repo, writes a first draft of every form field (title, tagline, about, key features, practice areas, demo URL), captures product screenshots from the live demo or a local dev server, and outputs a ready-to-paste submission folder. Use when the user wants to submit, showcase, or list a project on vibecode.law, or asks to draft a vibecode.law listing from a GitHub link.
license: MIT
user_invocable: true
metadata:
  author: nberk
  version: "1.0.0"
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

### Step 4 — Capture Gallery screenshots (demo URL first, local fallback)
Target spec: **up to 10 images, min 400×225px (16:9), max 4MB each.** Aim for
4–6 strong shots, not 10 filler ones.

Load the browser tools in one call:
`ToolSearch select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__resize_window,mcp__claude-in-chrome__read_page`

**A. If a demo URL exists:** open a new tab, `resize_window` to a 16:9 viewport
(e.g. 1600×900), navigate, and screenshot the landing view plus 2–5 key screens
(click into the main flows first so the shots show the product working, not an
empty state). Save each into `gallery/`.

**B. No demo URL → run it locally:** detect the dev command from `package.json`
`scripts` (`bun run dev` / `npm run dev`) or the framework default; start it in
the background; detect the port (parse the startup log, else probe 3000 / 4321 /
5173 / 8080); then capture as in A against `http://localhost:<port>`. Stop the
server when done. If it binds IPv6-only and the browser can't connect, retry with
the host bound to `0.0.0.0`. If it will not build or run, go to C.

**C. Neither works:** skip the Gallery. Record in the draft that screenshots must
be added manually, and why.

After capture, normalize with macOS `sips`: ensure each is ≥400×225, convert to
PNG/JPEG, and downscale anything over 4MB. Drop any shot smaller than the minimum.

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
