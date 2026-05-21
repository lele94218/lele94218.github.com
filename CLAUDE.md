# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hugo blog (Chinese, zh-cn) deployed to [blog.terryx.com](https://blog.terryx.com/) via GitHub Actions. Theme is [vinooganesh/hugo-ink](https://github.com/vinooganesh/hugo-ink) loaded as a git submodule.

## Commands

```bash
# Dev server
hugo server -D

# Build (mirrors CI)
hugo --gc --minify

# Clone (must include submodules for theme)
git clone --recursive https://github.com/lele94218/lele94218.github.com.git
```

Hugo version in CI: **0.161.1 extended**. No other build tools (no npm, no Makefile).

## Deployment

Push to `main` triggers `.github/workflows/deploy.yml` which builds and deploys to GitHub Pages. The workflow uses `submodules: recursive` to fetch the theme. Custom domain is set via `static/CNAME`.

## Architecture

### Layout Override Pattern

The project overrides specific theme layouts — Hugo resolves project layouts before theme layouts:

| Project Override | Purpose |
|---|---|
| `layouts/_default/baseof.html` | Adds chroma CSS, custom CSS, KaTeX (conditional on `math: true` frontmatter) |
| `layouts/index.html` | Homepage with full-width banner image + overlay title |
| `layouts/partials/header.html` | Simplified site title ("Home"), nav menu |
| `layouts/partials/footer.html` | Custom footer attribution, inline JS for theme toggle/scroll-top/code-copy |

Everything else (single.html, list.html, terms.html, 404, RSS) comes from `themes/ink/layouts/`.

### CSS Layering

1. **`themes/ink/assets/css/main.css`** — Base theme styles, CSS custom properties (`--bg`, `--text`, `--link`, etc.), Inter font
2. **`assets/css/chroma.css`** — Syntax highlighting. Uses `[data-theme="light"]` and `[data-theme="dark"]` selectors (light: colorful style, dark: github-dark style)
3. **`assets/css/custom.css`** — Hero banner styles, layout tweaks

All project CSS is processed through Hugo Pipes (minify + fingerprint) in baseof.html.

### Theme Switching

Default is **dark** (`data-theme="dark"` on `<html>`). JavaScript in footer.html toggles the attribute and persists to localStorage. Both chroma.css and main.css scope their color values to `[data-theme]` selectors — any new theme-aware CSS must follow this pattern.

### Content

- Posts go in `content/posts/` as Markdown with YAML frontmatter (`title`, `date`, `tags`, optional `math: true`)
- `content/about.md` — About page
- `hugo.toml` defines menus, taxonomies (tags/categories), related content settings
- Goldmark renderer has `unsafe = true` (allows raw HTML in Markdown)
- Chroma syntax highlighting uses `noClasses = false` (class-based, styled by chroma.css)

### Key Constraints

- Theme `h1-h4` font-family is locked to `"Iowan Old Style", Palatino, Georgia...` in main.css — overriding requires `!important`
- `.hero` section has significant default padding in main.css — custom.css zeroes top padding
- Banner title "欲說還休" uses traditional Chinese characters — font selection must support TC glyphs
- Do NOT push without user confirmation — always let user verify locally first
