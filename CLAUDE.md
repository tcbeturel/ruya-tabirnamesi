# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rüya Tabirnamesi is a single-page Turkish dream interpretation web app deployed on Vercel. It uses Claude as the AI backend via a serverless proxy function.

## Deployment

```bash
vercel --prod --yes        # deploy to production
vercel env add ANTHROPIC_API_KEY production --value "sk-ant-..." --yes  # set API key
```

The app is live at **https://ruya-tabirnamesi.vercel.app**

There are no build steps, no package.json, and no test suite — it's a static HTML file + one serverless function.

## Architecture

```
api/chat.js        — Vercel serverless function; proxies POST /api/chat → Anthropic API
public/index.html  — Entire frontend: SPA with 4 screens (ana/profil/arsiv/yorum)
vercel.json        — Security headers (CSP, X-Frame-Options, etc.) and routing
```

**Request flow:** Browser → `POST /api/chat` → `api/chat.js` → Anthropic API. The `ANTHROPIC_API_KEY` environment variable lives only on Vercel; it never reaches the browser.

**Frontend state** is stored entirely in `localStorage`:
- `ruya_arsiv` — array of dream sessions (capped at 50 entries)
- `ruya_profil` — user name and birth date for numerology

**XSS protection:** All user-supplied strings pass through `kacir()` (HTML escape) before being written to the DOM via `innerHTML`. The `formatla()` function calls `kacir()` first, then applies `**bold**`/`*italic*` markdown via regex.

## Key Constraints

- `api/chat.js` uses ESM (`export default`). Vercel auto-compiles it to CommonJS — do not add `"type": "module"` to a package.json unless you test the build impact.
- The CSP in `vercel.json` allows `'unsafe-inline'` for scripts and styles because all JS/CSS is inline in `public/index.html`. If you ever extract JS to an external file, update `script-src` to use a nonce instead.
- The frontend calls `/api/chat` (relative URL), so it only works when served from Vercel. Opening `public/index.html` directly from the filesystem will fail with network errors.
