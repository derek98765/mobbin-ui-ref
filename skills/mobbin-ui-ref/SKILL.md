---
name: mobbin-ui-ref
description: Find best-matching Mobbin UI references for a screen or user flow the user is working on, save full-quality reference images into the project's ref folder, and generate an HTML gallery explaining why each was picked and which is recommended. Use when the user wants UI/UX inspiration, reference screens, or "how do other apps do X" for a specific screen or flow they're building.
version: 1.0.0
user-invocable: true
argument-hint: "[screen or flow description, e.g. \"login screen\" or \"onboarding flow\"]"
---

Find the best-matching Mobbin references (5 by default) for the screen or flow the user is currently working on, save the images at full quality in an identifiable folder structure, then generate a consistent HTML gallery report with reasoning and a top recommendation.

## Step 1 — Determine what to search for

Figure out the target screen/flow from the user's message or current conversation context (e.g. a component being built, a Figma frame, a task description). If it's ambiguous — could be a single screen or a multi-step flow, or the platform (iOS vs web) is unclear — ask before searching.

If the user didn't specify how many references they want, ask before searching. Default to 5 if they have no preference. This count (call it N) determines the result limit in Step 2, the number of saved images in Step 3, and the grid size in Step 4.

Pick the right Mobbin tool:
- **Single screen** (e.g. "login screen", "empty state", "settings page") → `mcp__mobbin__search_screens`
- **Multi-step journey** (e.g. "onboarding", "checkout") → `mcp__mobbin__search_flows`
- **Website section** (e.g. "pricing section", "footer") → `mcp__mobbin__search_sections`

Write a specific, single-intent query describing the UI elements/steps you'd expect to see — not vague style words. Name a specific app only if the user asked to reference one directly.

This determines the **mode** for the rest of the skill:
- **Screen mode** (`search_screens` or `search_sections`): each of the N results is one standalone image.
- **Flow mode** (`search_flows`): each of the N results is itself an ordered *sequence* of consecutive screens (a full journey, e.g. all the steps of one app's onboarding). N is the number of flow references selected, not a raw screen count — each flow reference contributes its own multi-screen sequence.

## Step 2 — Select the N best matches

Request more results than N (e.g. `limit: N*2`, capped at the tool's max) so there's room to choose. Look at the actual returned images — not just metadata — and pick the N that most closely match the target screen/flow's purpose and key elements. Discard near-duplicates from the same app.

For each of the N, note:
- App name and platform
- The `mobbin_url` (required — every card must link back to it)
- A one-to-two sentence reason **specific to this reference** — what pattern, layout, or interaction makes it relevant to the target (not generic praise)
- **Screen mode:** the image URL to download.
- **Flow mode:** the ordered list of per-screen preview image URLs Mobbin returned for that flow (its consecutive steps), plus a short label for each step (e.g. "1. Welcome", "2. Sign up", "3. Personalize"). Keep Mobbin's own screen order — do not reorder or skip steps.

Decide which one is the **strongest overall recommendation** and why.

## Step 3 — Save reference images at full quality

Create this structure at the project root (create folders if they don't exist).

**Screen mode** — one image per reference:
```
ref/mobbin-refs/<screen-or-flow-slug>-<YYYY-MM-DD>/
  01-<appname>-<screenname>.webp (or .jpg, matching the source format)
  02-<appname>-<screenname>.webp
  ...
  0N-<appname>-<screenname>.webp
```

**Flow mode** — one subfolder per flow reference, containing its ordered screen sequence:
```
ref/mobbin-refs/<screen-or-flow-slug>-<YYYY-MM-DD>/
  01-<appname>-<flowname>/
    01-<stepname>.webp
    02-<stepname>.webp
    ...
  02-<appname>-<flowname>/
    01-<stepname>.webp
    ...
  ...
  0N-<appname>-<flowname>/
```

- `<screen-or-flow-slug>` is a kebab-case slug of what was searched (e.g. `login-screen`, `onboarding-flow`).
- Rank-prefix each top-level item (`01-`, `02-`, …) so ordering is visible in a file browser, with the top recommendation as `01-`. In flow mode, also rank-prefix each screen within a flow subfolder in Mobbin's step order (`01-`, `02-`, …).
- Slugify app, screen, and step names into filenames (lowercase, hyphens, no spaces/special chars).
- Download every image at the URL Mobbin returned as-is — **do not** run it through any compression, resizing, or optimization step (no `cwebp`, no `sharp`, no `/image-optimization` skill). Use `curl -sL -o <path> "<url>"` or equivalent. This preserves full source quality.
- If `image_format: webp` download fails or the client can't handle it, retry the same search with `image_format: jpg`.

## Step 4 — Generate the HTML gallery

Use `references/report-template.html` in this skill's directory as the **exact** template — same CSS custom properties, class names, and HTML structure every time. Do not redesign it per-run; only fill in content.

Fill in:
- Masthead: target screen/flow description, platform, reference count (N), today's date
- Recommended strip: the top pick's image (relative path into the saved subfolder), app name, platform, and a punchier restatement of why it's the best fit
  - **Flow mode:** use the top pick's first step image as the strip's representative image.
- Reference grid: one `.ref-card` per reference in ranked order (`01`–`0N`), each with:
  - Screen/app name, platform tag
  - The specific reason it was picked
  - A link to the `mobbin_url`
  - `is-recommended` class + `ref-star-badge` span added only to the top pick's card (see template comments)
  - **Screen mode:** `<img>` pointing to the **relative path** of the locally saved image (not the remote Mobbin URL) so the HTML works offline.
  - **Flow mode:** replace the single `<img>` with a horizontally-scrollable filmstrip of `<img>` tags, one per saved step in order, each with a small step-number/label caption — so the sequence of screens is visible at a glance. Keep the same card chrome (rank badge, star badge, title/app/reason/link) around the filmstrip.

Save the HTML as `ref/mobbin-refs/<screen-or-flow-slug>-<YYYY-MM-DD>/gallery.html`, inside the same subfolder as its images, so the relative image paths resolve correctly and the whole folder is portable.

## Step 5 — Report back

Tell the user:
- The folder path where images + gallery were saved
- That they can open `gallery.html` directly in a browser
- The top recommendation and a one-line reason

Do not print all N reasons again in chat — the HTML is the deliverable.

## Notes

- This skill only reads from Mobbin and writes to `ref/` — it does not modify any project source files, so no design-token or component-reuse rules apply here.
- `ref/` is a scratch location; do not commit its contents (respect the project's `.gitignore` — do not add a new ignore rule yourself without asking, per this project's existing rule on not touching `.gitignore`).
- If Mobbin returns fewer than N good matches for a narrow query, broaden the query once rather than padding with irrelevant results; if still short, tell the user how many were found and why.
