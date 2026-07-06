# WellScript Site Operations

**Purpose:** Operational runbook for the WellScript Hugo site. Commands, structure, encoding rules, and fixes for problems this site has actually had.

**Last updated:** July 5, 2026

---

## 1. The Basics (What This Site Is)

| Item | Value |
|---|---|
| Stack | Hugo static site generator + PaperMod theme |
| Repo root | The folder containing `content\` and the Hugo config (wherever cloned locally) |
| Live site | https://wellscript.io |
| Deploy | PowerShell publish script → GitHub → GitHub Pages |
| Local preview | http://localhost:1313 (only while `hugo server` is running) |

Hugo takes the markdown files in `content\` and builds them into HTML. Nothing is live until published. Local preview never touches the live site.

---

## 2. The Commands You Always Forget

**Preview the site locally (most common task).** From the repo root:
```powershell
hugo server
```
- Opens a dev server at **http://localhost:1313**
- Live-reloads when you save any file — leave it running while editing
- **Ctrl+C** to stop it
- Include draft posts with `hugo server -D`

**Build without serving (rarely needed manually):**
```powershell
hugo
```
Outputs static files to `public\`. Mostly useful as a quick check for build errors before publishing.

**Publish to the live site:**
Run the standard WellScript publish script. Give the deploy a few minutes, then verify on wellscript.io.

---

## 3. Content Structure (Why 404s Happen)

```
content\
├── _index.md          ← homepage front matter
├── about\index.md     ← About page (single page)
├── rmf\
│   ├── _index.md      ← section landing page (title + description)
│   └── some-post.md   ← articles
├── emass\_index.md
├── poam\_index.md
├── splunk\_index.md
└── resources\_index.md
```

**The rule that causes section 404s:** Hugo only builds a section page if the folder exists in `content\` with an `_index.md` in it. A menu link pointing at a section with no `_index.md` = 404. When adding a new nav item, create the matching `content\<section>\_index.md` first.

**`_index.md` vs `index.md`:**
- `_index.md` (underscore) = a *section* landing page that lists the posts inside it
- `index.md` (no underscore) = a normal *single* page (like About)

---

## 4. Front Matter Templates

**Section landing page (`_index.md`):**
```yaml
---
title: "Splunk"
description: "Practical guidance on Splunk in federal environments — ..."
---
```

**Single page like About (`index.md`):**
```yaml
---
title: "About"
description: "Who runs WellScript and why it exists — ..."
hideMeta: true
---
```
`hideMeta: true` hides the reading-time/byline row so nav pages look uniform. Articles should NOT have it — reading time and byline belong on articles.

**Articles** follow the WellScript Article Agent Prompt / Production Reference standard (date, tags, categories, cover image, etc.). That document remains the authority for article front matter.

---

## 5. Encoding Rules (The Mojibake Problem)

**The standard: every file is UTF-8 WITHOUT BOM.** Violating this is what caused `â€"` to appear on the live site where an em dash (—) should be.

**How to recognize it:**
- `â€"`, `â€™`, or similar garbage on a page = a file was written with the wrong encoding (mojibake)
- Windows PowerShell 5.1's `Out-File` and `Set-Content` default to the wrong encoding — **never use them for site files**
- PS 5.1 also reads `.ps1` files without a BOM as ANSI, so special characters *inside scripts* get mangled too. Fix: build special characters from char codes in scripts, e.g. `$em = [char]0x2014`

**The only safe write pattern:**
```powershell
$utf8 = New-Object System.Text.UTF8Encoding($false)   # $false = no BOM
[System.IO.File]::WriteAllText($path, $text, $utf8)
```

**Scan the whole site for BOM/mojibake damage.** From the repo root:
```powershell
Get-ChildItem ".\content" -Recurse -Filter *.md | ForEach-Object {
  $b = [System.IO.File]::ReadAllBytes($_.FullName)
  $r = [System.IO.File]::ReadAllText($_.FullName)
  if ($b.Length -ge 3 -and $b[0] -eq 0xEF -and $b[1] -eq 0xBB -and $b[2] -eq 0xBF) { Write-Host "BOM: $($_.FullName)" -ForegroundColor Red }
  if ($r -match "\u00E2\u20AC") { Write-Host "Mojibake: $($_.FullName)" -ForegroundColor Red }
}
Write-Host "No red lines = clean." -ForegroundColor Green
```

---

## 6. Running Downloaded PowerShell Scripts

The three errors you'll hit and their fixes:

**"is not recognized as the name of a cmdlet"** — wrong working directory (admin PowerShell starts in `C:\WINDOWS\system32`). Fix:
```powershell
cd "$env:USERPROFILE\Downloads"
```

**"running scripts is disabled on this system"** — execution policy. Fix (current session only, resets when window closes):
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
```

**Script runs but produces garbage characters** — the `.ps1` had special characters and PS 5.1 misread them (see Section 5). Rewrite using char codes.

**Also useful:** `Unblock-File .\script.ps1` clears the "downloaded from the internet" flag.

---

## 7. Standard Workflow: Publishing a New Article

1. Write the article per the Production Reference (front matter, UTF-8 no BOM, cover image)
2. Save it into the right section: `content\rmf\`, `content\splunk\`, or `content\poam\`
3. `hugo server` → check it at localhost:1313 (formatting, em dashes, cover image, section listing)
4. Ctrl+C, run the encoding scan from Section 5 if anything looked odd
5. Run the publish script
6. Verify on wellscript.io after the deploy finishes
7. Post the LinkedIn draft

---

## 8. Quick Troubleshooting Table

| Symptom | Cause | Fix |
|---|---|---|
| 404 on a section URL | No `_index.md` in that content folder | Create `content\<section>\_index.md` |
| `â€"` garbage on a page | File written with wrong encoding | Rewrite file with WriteAllText + UTF8 no BOM (Sec. 5) |
| Section page has no description | Missing `description:` in `_index.md` | Add it to front matter |
| Page shows reading time/byline where it shouldn't | Meta not hidden on a nav page | Add `hideMeta: true` |
| `hugo` not recognized | Hugo not on PATH in this window | Reopen terminal, or use full path to hugo.exe |
| Changes not on live site | Only previewed locally | Run the publish script; local server ≠ deploy |
| Script "not recognized" | Wrong working directory | `cd` to where the script actually is |
