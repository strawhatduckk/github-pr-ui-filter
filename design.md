# GitHub PR Comment Filter — Design Document

**Date:** 2026-03-06
**Status:** Draft

## Problem

GitHub PR pages become cluttered with bot and CI comments, making it hard to find human feedback. On a typical PR (e.g., [mozart#50925](https://github.com/padlet/mozart/pull/50925)), ~80% of comments come from bots:

| Author | Type | Count | Content |
|--------|------|-------|---------|
| `github-actions` | Bot | 5 | CI failure reports, automated code review |
| `neetoplaydash` | Bot | 3 | Replay/test recordings |
| `graphite-app` | Bot | 1 | Automation rules |
| `chatgpt-codex-connector` | Bot (review) | 1+ | AI code review |
| `cursor` | Bot (review) | 1+ | AI code review |
| `strawhatduckk` | Human | 1 | Stack comment |
| `xfatedky` | Human (review) | 1 | Human review |

This causes:
1. Hard to find human comments among bot noise
2. PR page feels cluttered and overwhelming
3. Important feedback gets missed / buried

## Solution

A **Tampermonkey/Violentmonkey userscript** that injects a floating filter panel on GitHub PR pages. Users can toggle comment visibility by author, with bots hidden by default.

### Why a userscript (not a browser extension)?

- No app store review process — iterate and share instantly via a Gist URL
- Tampermonkey is widely used — most devs have it or can install it in seconds
- Can graduate to a full extension later if needed
- The floating UI is identical either way — just injected DOM

## UX Design

### Collapsed state

A small floating pill in the bottom-right corner of the page:

```
┌──────────┐
│ 🔍 2/12  │
└──────────┘
```

Shows `{visible}/{total}` comment count. Clicking expands the panel.

### Expanded panel

```
┌─────────────────────────────┐
│  PR Comment Filter     [−]  │
│─────────────────────────────│
│  ⚡ Humans only │ All │ Bots │
│─────────────────────────────│
│  Bots                       │
│  ☐ github-actions  (5)      │
│  ☐ neetoplaydash   (3)      │
│  ☐ graphite-app    (1)      │
│  ☐ chatgpt-codex   (1)      │
│  ☐ cursor          (1)      │
│─────────────────────────────│
│  Humans                     │
│  ☑ strawhatduckk   (1)      │
│  ☑ xfatedky        (1)      │
│─────────────────────────────│
│  Showing 2/12 comments      │
└─────────────────────────────┘
```

- **Quick-action row:** "Humans only" (default), "All", "Bots only" — one-click presets
- **Per-author toggles** with comment count
- **Status bar** shows visible/total count
- Right-click an author → "Mark as bot" / "Mark as human" to override auto-detection

### Default behavior

- Bots start **unchecked** (hidden)
- Humans start **checked** (visible)
- First visit to any PR shows "Humans only" by default

## Technical Design

### Comment detection

The script targets two types of comments on the PR page:

1. **Timeline comments** — `.timeline-comment` containers in the Conversation tab. Author extracted from the author link element within each container.
2. **Inline review comments** — `.review-comment` containers in the Files Changed tab. Same author extraction approach.

On load, the script:
1. Scans all comment containers on the page
2. Extracts unique author usernames
3. Categorizes each as bot or human
4. Builds the filter panel
5. Applies default filters (hide bots)

### Filtering mechanism

When a toggle changes:
- Find all comment containers matching that author
- Set `display: none` (hidden) or `display: ''` (visible)
- Also hide corresponding timeline entry markers to avoid empty gaps
- Update the status bar count

### Lazy-loaded comments

GitHub lazy-loads comments on long PRs. The script uses a `MutationObserver` on the timeline container to:
- Detect newly added comment DOM nodes
- Extract their author
- Apply the current filter state automatically
- Update the author list in the panel if a new author appears

### Bot detection

**Automatic heuristics:**
- Username ends with `[bot]` → bot
- Username ends with `-app` → bot
- Username contains `-actions` → bot
- Username matches a hardcoded known-bots list

**Known bots list (initial):**
```
github-actions, graphite-app, neetoplaydash, codecov, netlify, vercel,
dependabot, renovate, cursor, chatgpt-codex-connector, coderabbit,
copilot, linear, mergify, percy, chromatic, snyk, sonarcloud
```

**Manual overrides:**
- Right-click context menu on any author in the panel
- "Mark as bot" / "Mark as human"
- Overrides persist via localStorage

### Persistence

Minimal persistence — only bot/human overrides are stored. Filter state always resets to "Humans only" on each PR page load.

| Key | Scope | Content |
|-----|-------|---------|
| `pr-filter-bot-overrides` | Global | `{ "username": "bot" \| "human" }` map of manual overrides |

- No per-repo or per-PR state — every PR starts fresh with "Humans only" default
- Bot/human overrides are global (a bot is a bot everywhere)

### Installation & distribution

- Hosted as a **GitHub Gist** with a raw URL
- Team members install by pasting the Gist raw URL into Tampermonkey's "Install from URL"
- Userscript headers include `@updateURL` and `@downloadURL` pointing to the Gist for auto-updates
- `@match` pattern: `https://github.com/*/pull/*`

### Edge cases

| Scenario | Handling |
|----------|----------|
| GitHub "Resolve conversation" | Script respects already-collapsed threads, doesn't interfere |
| All comments hidden | Shows subtle banner: "All comments filtered — adjust filters to see more" |
| Filtered inline comments (Files tab) | Hides the entire review thread block, not individual comments |
| Page navigation (SPA) | Listen for `turbo:load` / `pjax:end` events to re-initialize on navigation |
| No comments on PR | Panel pill shows `0/0` and panel body shows "No comments on this PR" |

## File structure

```
github-pr-comment-filter.user.js   # Single userscript file
```

Everything lives in one file — no build step, no dependencies.

## Future considerations

- Could graduate to a Chrome extension if the team wants a toolbar icon or cross-browser sync
- Could add keyboard shortcuts (e.g., `Shift+F` to toggle the panel)
- Could support filtering by comment type (CI failures, code review, deploy previews) in addition to author
