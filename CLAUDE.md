# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **GitHub profile README** repo: the repo name (`MarouaBoud`) matches the account name, so `README.md` renders as the landing page at https://github.com/MarouaBoud. It is not an application — there is no build system, no dependencies, no tests, and nothing to install. Two tracked files on `main`:

- `README.md` — the entire product
- `.github/workflows/snake.yml` — generates the contribution-graph snake animation

`.github/` is a dotfolder, so plain `ls` and Finder hide it. Use `ls -a`, `find .github -type f`, or `git ls-files`.

## The two-branch architecture

This is the one thing that is not discoverable from reading either file alone.

```
main  ──push──▶ snake.yml ──▶ dist/*.svg ──force-push──▶ output branch
                                                            │
README.md ◀── raw.githubusercontent.com/.../output/*.svg ───┘
```

- `main` holds source only. The generated SVGs **never exist on `main`**.
- `output` is a deploy branch holding only the two generated SVGs, force-pushed by `crazy-max/ghaction-github-pages` from `dist/`. Never commit to it by hand — the next run overwrites it.
- Because the SVGs live on another branch, `README.md` references them by absolute `raw.githubusercontent.com/MarouaBoud/MarouaBoud/output/...` URLs, **not** relative paths. A relative path would 404.
- Two variants are produced: `github-contribution-grid-snake.svg` (light) and `github-contribution-grid-snake-dark.svg` (via the `?palette=github-dark` suffix in the workflow's `outputs:` block). `README.md` picks between them with `<picture>` + `prefers-color-scheme`.
- **Renaming a workflow output requires updating three URLs** at the bottom of `README.md`: the dark `<source>`, the light `<source>`, and the `<img>` fallback.

Triggers: 12-hour cron (`0 */12 * * *`), every push to `main`, and manual dispatch. The job needs `permissions: contents: write` to push the `output` branch.

## Commands

There is no lint/build/test cycle. Verification is visual and happens after push.

```bash
gh workflow run snake.yml                    # regenerate the snake now
gh run list --workflow=snake.yml --limit 5   # check recent runs
gh run watch                                 # follow the current run

git fetch origin output                      # inspect generated artifacts
git ls-tree -r origin/output --name-only
```

No local preview is faithful: every visual element is a server-rendered request to a third-party service, and GitHub's own markdown sanitizer only applies on github.com. The real check is pushing to `main` and loading the profile page.

## Rendering constraints

- **GitHub sanitizes HTML in markdown.** `<div align>`, `<table>`, `<td width valign>`, `<details>/<summary>`, `<picture>`, and `<img width height src>` all work. `<style>`, `<script>`, `class`, and `style` attributes are stripped silently — layout must be done with the legacy presentational attributes already used in the file.
- **Markdown inside an HTML block needs blank lines around it.** This is why every `<td>` in the "Currently shipping" table has a blank line before and after its content. Removing those blank lines makes the markdown render as literal text.
- **Images are proxied and cached by GitHub's Camo.** An updated badge or SVG can appear stale for a while — that is caching, not a broken workflow. Confirm against the run log before "fixing" anything.
- **Every visual is an external dependency** and can rate-limit or go down: `shields.io`, `capsule-render.vercel.app`, `readme-typing-svg.demolab.com`, `github-readme-stats.vercel.app`, `streak-stats.demolab.com`, `github-profile-trophy.vercel.app`, `komarev.com` (view counter).
- **Header/typing text is URL-encoded query-string data**, not markdown. In the `capsule-render` and `readme-typing-svg` URLs, spaces are `%20`, `×` is `%C3%97`, and `—` is `%E2%80%94`. Edit these as URL params — a raw space or `&` breaks the whole image.

## Design system

Keep these consistent when adding anything visual:

| Token | Value | Used for |
|---|---|---|
| Primary | `7C5CFC` (purple) | Gradients, primary badges, links |
| Accent | `00e5a0` (green) | Gradient endpoint, secondary badges |
| Badge style | `flat-square` | All badges except the Connect section |
| Connect style | `for-the-badge` | Bottom Connect section only |
| Stats theme | `tokyonight` + `hide_border=true` | All stats/streak/trophy cards |

Gradients are always `color=0:7C5CFC,100:00e5a0`. One-off accents (`ff6b6b`, `f5a623`, `D4537E`) are reserved for award badges.

## Content conventions

- Sections are separated by `---` and led by an emoji `##` heading.
- `<details open>` for sections meant to be visible by default (hackathons, production systems, FinVerifyAI); plain `<details>` for deeper/older material (data engineering, secondary projects).
- "Currently shipping" is a fixed three-column `<table>` with `width="33%"` cells — adding a fourth project means restructuring, not appending.
- Project entries pair a bold linked title, a `>` blockquote one-liner, a prose detail paragraph, and inline-code tech tags.
