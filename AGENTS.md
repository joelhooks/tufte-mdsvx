# Agent instructions for `joelhooks/tufte-mdsvx`

This repo is a reusable wzrrd report template. Treat it as a clean public template, not a dump of whatever private report inspired it.

## Mission

Build narrative-first static reports:

1. **What the reader needs to know** — the story, decision, risk, or ask.
2. **Why we believe it** — receipts in links, margin notes, figures, or appendices.
3. **What happens next** — clear ownership and next action.

Do not make dashboard soup. Do not make a wall of cards. Do not bury the point under receipts.

## Hard rules

- Keep the template public-safe. No secrets, tokens, private paths, customer data, raw transcripts, or private operational details.
- Replace sample content before publishing a serious page.
- Keep paragraph ids stable once feedback may exist.
- Increment `pageRef.version` when paragraph text or ids materially change.
- Keep `feedbackParagraphs` synced with every `FeedbackParagraph` id that should export.
- Commit both D2 source and generated SVG for diagrams.
- Run `bun run check && bun run build` before publishing or claiming the template works.
- Publish the static `build/` directory with `wzrrd`; do not publish source directories as review artifacts unless explicitly requested.

## Wzrrd usage

Preferred local-template flow when `joel/tufte-mdsvx` is registered:

```bash
wzrrd init --template joel/tufte-mdsvx --slug <slug> --title "<title>" --emoji "🧙"
cd <slug>
bun install
bun run check
bun run build
wzrrd publish --file ./build --slug <slug> --expires-in 7d
```

Direct repo flow:

```bash
gh repo clone joelhooks/tufte-mdsvx <slug>
cd <slug>
bun install
# edit src/routes/+page.svx
bun run check
bun run build
wzrrd publish --file ./build --slug <slug> --expires-in 7d
```

For long-lived public pages, remove or extend `--expires-in` only when the operator asks. Use expiring noindex review links by default.

## Authoring checklist

When making a report from this template:

1. Read this file and `README.md`.
2. Set the page identity in `src/routes/+page.svx`:
   - `pageRef.slug`
   - `pageRef.url`
   - `pageRef.version`
   - `pageRef.run`
3. Write the lead first. If the lead does not answer “why should the reader care?”, fix that before adding components.
4. Use `StatLine` only for tiny metadata, not fake KPI theater.
5. Wrap review-worthy paragraphs:

```svelte
<FeedbackParagraph id="risk-summary" storagePrefix={feedbackStoragePrefix} note="Receipt hint for the reviewer.">
  <p>Plain-language paragraph here.</p>
</FeedbackParagraph>
```

6. Add every exported paragraph id to `feedbackParagraphs`.
7. Use `MarginNote` for proof/caveat snippets:

```svelte
<MarginNote id="source-log" number="1">Receipt: command output, URL, screenshot, or short caveat.</MarginNote>
```

8. Keep margin notes short. If the receipt is huge, link it or summarize it.
9. Build diagrams as source plus generated asset:

```bash
d2 static/report-loop.d2 static/report-loop.svg
```

10. Run checks and publish.

## Feedback model

`FeedbackParagraph` stores local browser feedback under:

```text
<storagePrefix><paragraph-id>
```

Default storage prefix:

```text
tufte-mdsvx-feedback:
```

`FeedbackExport` copies a compact payload:

```json
{
  "type": "tufte_mdsvx_feedback",
  "ref": {
    "slug": "example-slug",
    "version": "0.1.0",
    "run": "example-run",
    "url": "https://example-slug.wzrrd.sh/"
  },
  "items": [
    { "p": "risk-summary", "r": "important", "s": "Make the ask clearer." }
  ]
}
```

Agents applying pasted feedback should:

1. Verify `ref.slug`, `ref.version`, and `ref.url` match the page/source they are editing.
2. Locate paragraphs by `p`, not by fuzzy text matching.
3. Apply `s` as steering, not as a command to blindly paste.
4. Treat `r` as a signal:
   - `good`: preserve the paragraph shape.
   - `sharp`: this wording/style worked; use more like it.
   - `important`: raise or clarify the point.
   - `confusing`: rewrite for clarity.
   - `off`: challenge/remove/reframe.
5. Update `pageRef.version` after edits.
6. Re-run `bun run check && bun run build`.

## Component intent

- `ReportLayout`: one-column report shell with room for desktop margin notes.
- `MarginNote`: Tufte-style sidenotes for source proof, caveats, and agent reasoning.
- `FeedbackParagraph`: reviewable paragraph wrapper with reactions and steering drawer.
- `FeedbackExport`: bottom export button for agent-readable feedback.
- `FlowChart`: static SVG figure wrapper, usually generated from D2.
- `PatternCards`: small supporting pattern/receipt cards. Use sparingly.
- `StatLine`: compact metadata. Keep it boring.

## Design taste

Use:

- one primary highlight color;
- low saturation;
- generous line-height;
- short sections;
- strong links;
- side notes for receipts;
- real prose.

Avoid:

- nested boxes;
- badge lasagna;
- huge card grids;
- corporate report sludge;
- unexplained metrics;
- raw log dumps in the main story;
- AI/system jargon unless the reader needs it.

## Verification

Before final handoff, report:

- files changed;
- commands run;
- published wzrrd URL if any;
- whether the repo/source was pushed;
- any warnings, especially Hugeicons/Vite barrel warnings;
- anything intentionally left unhandled.

Known non-fatal warning: Vite/Rolldown may warn that `@hugeicons/core-free-icons` is a large barrel module. It is acceptable for small review pages. If this template becomes a heavier app, add a transform-imports optimization or direct typed icon imports.
