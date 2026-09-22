# FunType

A typing-speed trainer. Open `index.html` in any browser — no server, no build, no install.

## Files

| File | What it is |
|---|---|
| `index.html` | **The app.** Edit this one. Self-contained: HTML + CSS + JS in one file. |
| `funtype.html` | Same page without the `<!doctype>/<html>/<head>/<body>` wrapper — the copy published as a Claude Artifact. Regenerate it after editing (see below). |
| `logo.svg` | Full lockup: keycap mark + wordmark. Needs Gabarito installed to render the text correctly. |
| `logo-mark.svg` | Icon only — pure geometry, safe anywhere (app icon, README, social avatar). |
| `logo-favicon.svg` | The mark retuned for 16–24px: bigger cap, thicker caret, less padding. **Use this below ~28px.** |
| `PROPOSAL.md` | Project proposal — scope, the measurement model and why it is hard to change, architecture, roadmap and risks. |

To regenerate the artifact copy after editing `index.html`, strip the first 5 lines and the last 2:

```bash
sed '1,5d' index.html | sed '$d' | sed '$d' > funtype.html
```

## Modes

- **time** — 15 / 30 / 60 / 120 seconds, words keep coming
- **words** — a fixed 10 / 25 / 50 / 100 words
- **text** — short / medium / long paragraphs, for sustained rhythm
- **code** — js / py / sql snippets, for symbol and shift-key drilling

Toggles: `! punctuation` adds capitals, commas, quotes and brackets; `# numbers` mixes digits in.

## Keys

| Key | Does |
|---|---|
| `tab` | restart — same words again |
| `esc` | new test — fresh words |
| `ctrl` + `backspace` | delete the whole current word |
| `backspace` | step back, including into a previous word you got wrong |
| `enter` | on the results screen, start the next test |

## How the numbers are worked out

- **net wpm** — correct characters ÷ 5 ÷ minutes. A correctly typed word also earns its trailing space; a word you left wrong does not.
- **raw wpm** — every character you typed, right or wrong, over the same clock.
- **accuracy** — keystrokes that hit the right character ÷ all keystrokes. Backspaces are free.
- **consistency** — how even your per-second pace was: `100 × (1 − stdev/mean)` across the run. High speed with low consistency means you are sprinting and stalling.
- **characters** — `correct / wrong / extra / missed`. *Extra* is typing past the end of a word, *missed* is hitting space too early.

## Progress

Personal bests are kept per exact configuration (`time-30`, `words-25-p`, `code-sql`…), so turning punctuation on starts a fresh record rather than polluting the old one. Everything lives in `localStorage` under `funtype.v1` — it stays on this browser, on this machine, and never leaves it.

## Design

- **Ink** `#16161e`, **accent apricot** `#ff9e64`, typed text `#e6e4f0`, errors `#f7768e`.
- Chart series `#d8792e` (net) / `#6189e0` (raw) / `#cf4a66` (errors) — checked for colour-blind separation against both the dark and light surfaces.
- Gabarito for the interface, JetBrains Mono for everything you actually type.
- Eight themes in the footer: dusk, daylight, matcha, cobalt, nordfall, bubblegum, mono, terminal.

The logo is a keycap whose legend is a text caret — the same caret you chase across
the screen while typing. It blinks on the same 1.05s beat as the real one, and the cap
face depresses 2.6px when you press it, because the brand is also the restart button.

The caret is `--accent-ink` sitting directly on the `--accent` cap, with no dark dish
behind it. That is the whole trick: two shapes instead of four, so the caret is still a
caret at 16px instead of collapsing into a smudge. Every theme defines an `accentInk`
that contrasts hard against its `accent`, so the mark re-colours cleanly through all
eight — checked against dusk, daylight, bubblegum and terminal.

`logo-favicon.svg` is the same drawing with the cap pushed out to the edges and the
caret thickened, for sizes where the 4px skirt and the thin serifs would silt up. Swap
at about 28px. The app carries it inline as a `data:` URI so `index.html` still opens
from anywhere with nothing beside it.

The wordmark is Gabarito ExtraBold, tracking -0.022em, `Fun` in accent and `Type` in
text. In `logo.svg` it is live text — outline it before handing the file to anything
that will not have Gabarito.
