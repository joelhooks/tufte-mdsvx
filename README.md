# Tufte MDSvX for wzrrd

A quiet SvelteKit + MDSvX template for narrative-first static reports published with `wzrrd`.

Use it for field notes, incident reports, project reviews, launch memos, agent run summaries, and any document where the story should lead while receipts stay nearby.

## What is in here

- SvelteKit static build via `@sveltejs/adapter-static`
- MDSvX page authoring in `src/routes/+page.svx`
- Tufte-style margin notes
- D2/SVG figure component
- Pattern cards and compact stat lines
- Paragraph-level feedback controls
- Bottom-of-page feedback export button for LLM/agent follow-up
- Agent instructions in `AGENTS.md`

## Quick start

```bash
bun install
bun run check
bun run build
```

Preview locally:

```bash
bun run preview
```

Publish with wzrrd:

```bash
wzrrd publish --file ./build --slug your-readable-slug --expires-in 7d
```

If your local wzrrd has this template registered as `joel/tufte-mdsvx`, initialize a new site with:

```bash
wzrrd init --template joel/tufte-mdsvx --slug your-readable-slug --title "Your title" --emoji "🧙"
cd your-readable-slug
bun install
bun run check
bun run build
wzrrd publish --file ./build --slug your-readable-slug --expires-in 7d
```

## Authoring loop

1. Edit `src/routes/+page.svx`.
2. Replace the sample page with the real report.
3. Keep `pageRef` accurate:
   - `slug`: the intended wzrrd slug
   - `url`: the final public URL
   - `version`: increment this whenever paragraphs materially change
   - `run`: the report/run id, date, issue id, or other anchor
4. Wrap important paragraphs in `FeedbackParagraph` with stable ids.
5. Keep `feedbackParagraphs` in sync with those ids.
6. Add source/caveat notes with `MarginNote`.
7. Put diagrams in `static/*.d2`, generate `static/*.svg`, and render them with `FlowChart`.
8. Run `bun run check && bun run build`.
9. Publish `./build` with `wzrrd`.

## Feedback payload

The copy button exports compact JSON like:

```json
{
  "type": "tufte_mdsvx_feedback",
  "ref": {
    "slug": "tufte-mdsvx-example",
    "version": "0.1.0",
    "run": "example-run",
    "url": "https://tufte-mdsvx-example.wzrrd.sh/"
  },
  "items": [
    { "p": "risk-summary", "r": "important", "s": "Make the ask sharper." }
  ]
}
```

Keys are intentionally short:

- `p`: paragraph id
- `r`: reaction
- `s`: steering text

Paste that payload back into an agent and it can find the exact paragraph without guessing.

## Public/private boundary

Before publishing a page, check that the generated bundle does not include:

- secrets, tokens, credentials, private keys
- private customer data
- raw transcripts or private chat logs
- local-only machine paths unless intentionally shared
- private repo/ops details that do not belong in the audience's view

Use links and margin notes for receipts, but do not leak the basement.

## Files to edit most often

```text
src/routes/+page.svx          # report prose and page metadata
src/app.css                   # theme and layout
src/lib/*.svelte              # reusable report components
static/report-loop.d2         # example diagram source
static/report-loop.svg        # generated diagram asset
AGENTS.md                     # agent rules for future sessions
```
