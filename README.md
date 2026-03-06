# GitHub PR Comment Filter

A Tampermonkey userscript that filters bot comments on GitHub PR pages. Hides bot noise, shows human feedback.

## The Problem

GitHub PR pages get cluttered with bot comments (CI reports, automated code review, deploy previews). On a typical PR, ~80% of comments come from bots, burying the human feedback that actually matters.

## What It Does

- Floating pill in the bottom-right corner showing visible/total comment count
- Expands to a panel with **Humans only** / **All** / **Bots only** quick presets
- Per-author checkboxes for fine-grained control
- Right-click any author to reclassify as bot or human (persisted across sessions)
- Auto-detects bots via heuristics and a known-bots list
- Handles lazy-loaded comments and SPA navigation

## Setup

### 1. Install Tampermonkey

- [Chrome](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)
- [Firefox](https://addons.mozilla.org/en-US/firefox/addon/tampermonkey/)
- [Edge](https://microsoftedge.microsoft.com/addons/detail/tampermonkey/iikmkjmpaadaobahmlepeloendndfphd)

### 2. Enable Developer Mode (Chrome only)

1. Go to `chrome://extensions`
2. Toggle **Developer mode** ON (top-right corner)

### 3. Install the Script

1. Click the Tampermonkey icon in your toolbar → **Create a new script**
2. Delete the default content
3. Paste the contents of [`github-pr-comment-filter.user.js`](github-pr-comment-filter.user.js)
4. Press **Ctrl+S** (or **Cmd+S**) to save

### 4. Use It

Navigate to any GitHub PR page. You'll see the filter pill in the bottom-right corner.

## Bot Detection

Usernames are classified as bots if they:
- End with `[bot]` or `-app`
- Contain `-actions`
- Match the built-in known-bots list (github-actions, dependabot, codecov, etc.)

Right-click any author in the panel to override the classification. Overrides persist in localStorage.
