# Boundary pressure report

Use this format only for a broad repository or subsystem review with multiple candidate seams. The artifact helps the user compare candidates; it is not a substitute for evidence.

## Create the artifact

Write a self-contained HTML file to the OS temp directory, named `boundary-pressure-<timestamp>.html`. Use Mermaid via CDN for dependency and sequence graphs; use inline CSS or SVG for change-pressure and before/after diagrams. Keep the repository clean. Open the file with the platform's normal opener when permitted, and always provide its absolute path.

The page should be readable without interacting with it. Favor editorial clarity and generous whitespace over dashboard chrome.

## Report shape

Start with a one-screen **pressure map** of the reviewed area:

- boxes are modules, labeled by responsibility;
- solid arrows are public dependencies;
- red arrows are leaked implementation knowledge;
- a halo or heat tint marks reasons to change that spread across modules;
- a heavy shared outline marks one invariant or transaction spanning a seam.

Then render one card per candidate. Each card contains:

- **Verdict and strength** — Keep, Move, Narrow, Invert, Split, or Merge; Strong, Worth exploring, or Speculative.
- **Modules and files** — the concrete scope.
- **Observed pressure** — evidence from code, tests, history, or documented constraints.
- **Coupling ledger** — knowledge crossing the seam and the change it propagates.
- **Before / after** — side-by-side diagrams centered on one realistic change.
- **Change simulation** — compact touch-point lists for today and proposed design.
- **Tradeoff** — the cost or coupling that remains.

End with two short sections:

1. **Best first move** — the highest-leverage, lowest-speculation candidate.
2. **Seams to keep** — boundaries that were examined and proved meaningful. This prevents a review from reading like a refactor quota.

## Visual patterns

Choose the diagram that exposes the pressure:

- **Ripple map** — a change at one module radiates to every file or test it forces to move.
- **Knowledge leak** — annotate each arrow with the representation, control choice, or call order that crosses it.
- **Split strain** — one invariant stretches across two modules with repeated calls in both directions.
- **Responsibility magnet** — scattered behavior collapses around the module that actually owns it.
- **Contract shrink** — show the current payload or interface beside the smaller domain contract.

Diagrams carry the argument. Redraw any diagram that needs a paragraph to decode.

## Minimal scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Boundary pressure — {{repository}}</title>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      :root { --ink:#172033; --paper:#f7f5ef; --card:#fff; --leak:#c53b32; --keep:#27735b; --line:#c9c4b8; }
      * { box-sizing: border-box; }
      body { margin:0; background:var(--paper); color:var(--ink); font:16px/1.5 ui-sans-serif,system-ui,sans-serif; }
      main { width:min(1100px,calc(100% - 32px)); margin:0 auto; padding:56px 0; }
      article { margin:32px 0; padding:28px; background:var(--card); border:1px solid var(--line); border-radius:18px; }
      .pair { display:grid; grid-template-columns:1fr 1fr; gap:20px; }
      .leak { color:var(--leak); }
      @media (max-width:760px) { .pair { grid-template-columns:1fr; } }
    </style>
  </head>
  <body><main><!-- pressure map, candidate cards, recommendation --></main></body>
</html>
```

Keep prose compact. Name the precise knowledge that leaks; avoid vague conclusions such as “too coupled,” “cleaner,” or “more maintainable.”
