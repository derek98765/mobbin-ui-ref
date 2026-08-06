# mobbin-ui-ref

A Claude Code skill that finds real-app UI/UX references for a screen or
user flow you're about to build, using the Mobbin MCP. It saves full-quality
reference images into the current project and writes a browsable HTML
gallery page with a picked recommendation.

## What's in this repo

```
skills/
  mobbin-ui-ref/SKILL.md                      # the skill
  mobbin-ui-ref/references/report-template.html  # fixed gallery page template — read, not improvised, at runtime
  mobbin-ui-ref/doc/mobbin-docs.md            # Mobbin MCP reference notes
  mobbin-ui-ref/doc/openapi.json              # Mobbin API schema
```

Install by copying `skills/mobbin-ui-ref` into `~/.claude/skills/` (or a
project's `.claude/skills/`).

## Prerequisites

- **A Mobbin Pro plan subscription.** The Mobbin MCP server requires an
  authenticated Mobbin account, and the search tools this skill calls
  (`search_flows`, `search_screens`, `search_sections`) are gated behind a
  Pro plan. A free Mobbin account is not sufficient.
- The Mobbin MCP server registered and authenticated in Claude Code (see
  Setup below).
- A project directory to save references into — the skill writes to
  `ref/mobbin-refs/<screen-or-flow-slug>-<YYYY-MM-DD>/` at the project root.

## Setup — connecting the Mobbin MCP server (first time only)

The recommended way is through Claude Code's Settings UI (Settings →
Connectors → add Mobbin), which handles registration and auth in one flow.

If you need to do it via the CLI instead:

1. Register the Mobbin MCP server:
   ```bash
   claude mcp add mobbin --scope user --transport http https://api.mobbin.com/mcp
   ```
2. Start a new Claude Code session and open the MCP server list:
   ```bash
   claude /mcp
   ```
   Select **mobbin** from the list.
3. Select **Authenticate**.
4. A browser window opens automatically — sign in with your Mobbin account
   (Pro plan required) to authorize access.

Once authenticated, the `search_flows` / `search_screens` / `search_sections`
tools become available and this skill can use them.

## What it does and why

Before building a screen or flow, it's easy to either start from a blank
page or copy the first vaguely-similar example that comes to mind. This
skill turns "find me some references" into a structured, repeatable step:

1. **Scopes the request** — figures out the target screen or flow, whether
   it's a single screen (`search_screens`), a multi-step journey
   (`search_flows`), or a page section (`search_sections`), and the target
   platform (iOS vs. web). Asks before searching if either is ambiguous.
2. **Asks how many references** to find if not specified, defaulting to 5.
   This count (N) drives everything downstream.
3. **Searches Mobbin** with one specific, single-intent query, requesting
   more than N results so there's room to choose. Broadens the query once
   if results are thin, rather than firing off near-duplicate searches.
4. **Selects and saves the N best matches** at full, uncompressed quality
   into `ref/mobbin-refs/<slug>-<date>/`, rank-prefixed (`01-`, `02-`, …) so
   relevance order is visible in a file browser.
   - In **screen mode**, each reference is one saved image.
   - In **flow mode**, each reference is a whole journey — its own
     subfolder holding the ordered sequence of consecutive screens Mobbin
     returned for that flow, in step order.
5. **Picks a recommendation** — the single strongest overall match, with a
   concrete reason specific to that reference (not generic praise).
6. **Writes a consistent HTML gallery** (`gallery.html` in the same folder)
   using the bundled `report-template.html` — every run uses the exact same
   layout. Flow references render as a horizontally-scrollable filmstrip of
   their screen sequence; screen references render as a single image.
7. **Reports back** the folder path, that `gallery.html` can be opened
   directly in a browser, and the top recommendation with a one-line reason
   — the HTML is the deliverable, so it doesn't repeat all N reasons in chat.

## How to invoke

```
/mobbin-ui-ref [screen or flow description]
```

Or just ask in conversation — e.g. "find me references for the expense list
screen" or "look up onboarding flow examples on mobbin" — and Claude will
pick up the skill automatically based on its description.

## Example

Prompt: *"Find me 5 references for an add-expense screen, similar to
budgeting apps."*

- **Scope:** single screen, slug `add-expense`, platform `ios`.
- **Search:** `search_screens` with a concrete query (e.g. "add expense
  entry form with amount and category"), requesting more than 5 results.
- **Save:** 5 images saved to `ref/mobbin-refs/add-expense-2026-08-06/`:
  ```
  ref/mobbin-refs/add-expense-2026-08-06/01-ynab-add-expense.webp
  ref/mobbin-refs/add-expense-2026-08-06/02-cash-app-add-expense.webp
  ref/mobbin-refs/add-expense-2026-08-06/03-mint-add-expense.webp
  ref/mobbin-refs/add-expense-2026-08-06/04-copilot-add-expense.webp
  ref/mobbin-refs/add-expense-2026-08-06/05-monarch-add-expense.webp
  ```
- **Recommendation:** e.g. YNAB — "clear visual hierarchy for amount vs.
  category, and its pill-style category selector is easy to adapt."
- **Gallery:** `ref/mobbin-refs/add-expense-2026-08-06/gallery.html`, built
  from `report-template.html`, with the YNAB card marked "★ Recommended."
- **Report:** folder path, note that the gallery opens directly in a
  browser, and the top pick with a one-line reason.

For a flow (e.g. "onboarding flow references"), each of the N picks would
instead be its own subfolder of ordered screenshots, and its gallery card
would show a filmstrip of that flow's steps instead of one image.

## Notes

- Not for critiquing an already-decided design (use `critique`) or for
  implementing one (use `frontend-design` / `implement-design`) — this
  skill is for the reference-gathering step before either of those.
- `ref/` is a scratch location; the skill does not modify project source
  files and does not commit its output.
- The gallery template (`report-template.html`) is fixed on purpose — the
  skill reads it rather than generating markup from scratch, so every
  gallery stays visually consistent across runs.
