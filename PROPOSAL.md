# FunType — Project Proposal

*A browser-based typing-speed trainer. No server, no build, no install.*

2026-09-22 · Kaon

## Executive summary

**FunType is a typing-speed trainer that runs entirely in a browser tab.** One HTML file of roughly 45 KB carrying its own markup, styles and logic. No server, no build step, no install, no account, no network call except two font requests. Open the file and type.

A working build exists today at [github.com/KaonHew02/FunType](https://github.com/KaonHew02/FunType). It is not a prototype. Four drill modes, a complete metrics engine, per-configuration personal bests, eight themes and a finished brand mark are all shipped and working. What this document does is set down what was built, why each decision went the way it did, and what the next phases should be — so the project can be evaluated, handed to someone else, or picked up after a six-month gap without archaeology.

### What this proposal asks for

Not money and not headcount. It asks for a decision on three things:

1. **Whether to publish it properly.** A real URL instead of a file opened off disk. The repository already carries a `.nojekyll` marker, so GitHub Pages was anticipated but never switched on.
2. **Which roadmap phases in §7 are worth building.** Every one of them spends some of the single-file simplicity that makes the current build good. That trade should be made deliberately, not drifted into.
3. **Whether the measurement model in §4 is the one to commit to.** Personal bests become meaningless the moment the definitions shift underneath them, so this is the one decision that is expensive to reverse.

### The shape of the bet

Typing trainers are a crowded category and FunType does not win by out-featuring the incumbents. It wins on three things: honest measurement, instant start, and being pleasant to sit inside for twenty minutes. Each of those is cheap to hold and awkward for a larger product to copy, because each is a *refusal* — no account, no leaderboard, no telemetry — rather than a feature. Refusals are the part of a product a competitor cannot add.

## Problem and opportunity

### The problem

People who want to get faster at typing have plenty of places to practise and very few places to practise *honestly*. The category splits into two failure modes.

**The gamified trainers** — the ones aimed at schools — wrap the drill in badges, streaks, cartoon races and mandatory accounts. The measurement is usually gross WPM, which counts every keystroke whether or not it was correct, so the number climbs as you get sloppier. That is precisely backwards: it rewards the habit you are trying to break.

**The minimalist trainers** — the ones serious typists actually use — measure well but have drifted into being platforms. Accounts, leaderboards, friend lists, seasonal events, and a settings surface deep enough to need its own documentation. The drill is still in there, but you pass through a login and three menus to reach it, and your results are on someone else's server.

Both share a quieter flaw: **the metric moves without telling you.** Turn punctuation on and your WPM drops several points, so the personal best you set last week is no longer comparable to today's run. Most trainers keep one "best" number across every configuration, which makes it a record of which settings were easiest rather than of how fast you type.

### The opportunity

There is room for a trainer that does three things and refuses the rest:

- **Measures the way a skeptic would.** Net WPM as the headline, raw WPM beside it so the gap between them is visible, and a consistency score that exposes sprint-and-stall pacing. §4 sets out each definition.
- **Starts instantly.** No login, no splash, no mode-selection wizard. The words are on screen before you have decided to practise, and typing begins the test.
- **Keeps records that mean something.** Personal bests stored per exact configuration, so `time-30` and `time-30` with punctuation are separate records rather than one polluted number.

### Who it is for

The primary user is **someone who already types for a living** — developers, writers, support staff — who wants twenty focused minutes and a number they can trust, not a curriculum. The `code` mode exists specifically for this person: symbol and shift-key drilling on real JavaScript, Python and SQL, which is where a working developer's actual speed ceiling sits.

A secondary user is **the privacy-conscious learner** who does not want a typing account. Everything FunType records stays in `localStorage` on one browser, on one machine, and never leaves it. That is a genuine differentiator and costs nothing to maintain — it is what *not* building a backend gets you for free.

An explicit non-user is **the classroom**. Cohort management, assigned lessons and teacher dashboards are a different product with a different shape, and chasing them would destroy the properties above.

## Scope — what FunType does today

Everything in this section is built and working in the current build. Nothing here is aspirational.

### The four drill modes

| Mode | Amounts | What it trains |
| --- | --- | --- |
| **time** | 15 / 30 / 60 / 120 seconds | Sustained rate against a clock. Words keep coming until time runs out. |
| **words** | 10 / 25 / 50 / 100 words | A fixed target, so the run ends on an achievement rather than a buzzer. |
| **text** | short / medium / long | Real paragraphs, for rhythm across sentences rather than disconnected words. |
| **code** | js / py / sql | Symbol density and shift-key reach on real source. Where a working developer's ceiling actually sits. |

The `time` and `words` modes draw from an inline list of **298 common English words** (288 unique), averaging 4.4 characters. The list is weighted toward short, high-frequency words, with a deliberate tail of typing-specific vocabulary — *keyboard, symbol, accuracy, rhythm, finger, motion* — so the drill occasionally rehearses its own subject matter.

### Two toggles

- **`! punctuation`** adds capitals, commas, quotes and brackets.
- **`# numbers`** mixes digits into the stream.

Both change the difficulty materially, which is why both are part of the personal-best key rather than a display setting. See §4.

### Key bindings

| Key | Does |
| --- | --- |
| `tab` | Restart — the same words again |
| `esc` | New test — fresh words |
| `ctrl` + `backspace` | Delete the whole current word |
| `backspace` | Step back, including *into a previous word you got wrong* |
| `enter` | On the results screen, start the next test |

That backspace behaviour is worth calling out. Most trainers lock a word once you have passed it. FunType lets you reverse into a word you botched, which matches how a real editor behaves and keeps the accuracy figure meaningful rather than punitive.

### The results screen

On finishing, the test view is replaced by a results panel carrying:

- **Headline** — net WPM and accuracy, with a `new best · +N` badge when the run beats the stored record for that exact configuration, or `first record` when there was none.
- **A chart** — net WPM and raw WPM plotted per second, with error marks where mistakes landed. The axis snaps to a sensible maximum from a fixed ladder rather than scaling to the data, so two runs can be compared by eye.
- **Six facts** — test configuration, raw WPM, consistency, the character breakdown, elapsed time, and the personal best for that configuration.
- **Two buttons** — *next test* (fresh words) and *repeat this text* (same words, to isolate improvement from luck of the draw).

### Personal bests

Records are stored **per exact configuration**. The key is composed from mode, amount and the active toggles — `time-30`, `words-25-p`, `code-sql` — so switching punctuation on starts a fresh record rather than polluting the old one. A best is claimed on net WPM only; accuracy is stored alongside it for context but does not gate the record.

The header carries three live figures: best for the current configuration, all-time best across every configuration, and total tests run.

## Measurement model

This is the part of the product that has to be right, because it is the part users are trusting. Every definition below is implemented as stated.

### Net WPM — the headline number

```
net wpm = correct characters ÷ 5 ÷ minutes
```

Five characters is the long-standing convention for one "word", which keeps the figure comparable to every other trainer. The consequential choice is **which characters count**: a correctly typed word also earns its trailing space, and a word left wrong does not. So a run of sloppy words does not quietly inflate the headline.

### Raw WPM — shown beside it, deliberately

```
raw wpm = every character typed ÷ 5 ÷ minutes
```

Right or wrong, over the same clock. Raw is not a vanity metric here — it is displayed *next to* net specifically so the gap between the two lines is visible. A wide, growing gap is the signature of someone typing faster than their accuracy can support, which is the single most common way to plateau.

### Accuracy

```
accuracy = keystrokes that hit the right character ÷ all keystrokes
```

**Backspaces are free.** They are not counted as keystrokes and not counted as errors. This is a deliberate departure from trainers that penalise correction: the goal is to train accurate typing, not to train people into leaving mistakes uncorrected because fixing them costs score.

### Consistency

```
consistency = 100 × (1 − stdev / mean)
```

Computed across per-second samples of raw pace, clamped to 0–100. Each second the engine records how many characters were typed in that second and converts it to a WPM-equivalent; consistency is how even that series was.

This is the most diagnostic number on the screen and the least understood. **High speed with low consistency means sprinting and stalling** — bursts on familiar words, freezing on unfamiliar ones. Two typists with identical WPM and identical accuracy can have very different consistency, and the one with the lower score has the larger, easier gain available: they do not need to get faster, they need to stop stopping.

### The character breakdown

Reported as `correct / wrong / extra / missed`.

| Term | Means |
| --- | --- |
| **correct** | Hit the right character in the right place |
| **wrong** | Hit the wrong character |
| **extra** | Typed past the end of a word |
| **missed** | Hit space too early, leaving characters untyped |

Splitting *extra* from *missed* matters because they are opposite faults with opposite fixes. Consistent *missed* counts mean you are anticipating the word boundary; consistent *extra* counts mean you are not reading ahead far enough.

### Why this model is hard to change later

Every stored personal best was produced under these exact definitions. Changing any of them — counting the trailing space differently, charging for backspaces, altering the consistency window — silently invalidates every record a user holds, with no way to recompute them because the raw keystroke history is not retained.

**This is the one decision in the proposal that is expensive to reverse.** If the model is to change, it should change before publication, and the storage key should be versioned from `funtype.v1` to `funtype.v2` so old records are retired rather than misread.

## Technical architecture

### The central constraint

**One file, no build.** `index.html` is 45,068 bytes and contains the markup, the stylesheet and the whole application in a single document. There is no bundler, no package manager, no `node_modules`, no transpile step and no framework. Vanilla JavaScript against the DOM.

This is a constraint, not an accident, and it buys four things:

- The app opens from a `file://` path, a USB stick, an email attachment or a web host, identically.
- There is no toolchain to rot. A build that works today works in five years, because there is nothing to reinstall.
- The whole program is readable in one sitting, which is why the measurement model in §4 can be audited rather than trusted.
- Deployment is a file copy.

The cost is real and is priced into §7: every roadmap phase that adds meaningful surface area puts pressure on the single-file rule.

### Files

| File | What it is |
| --- | --- |
| `index.html` | **The app.** The one to edit. |
| `funtype.html` | The same page minus the doctype/html/head/body wrapper — the copy published as a Claude Artifact. Regenerated, never hand-edited. |
| `logo.svg` | Full lockup: keycap mark plus wordmark. Needs Gabarito to render the text correctly. |
| `logo-mark.svg` | Icon only, pure geometry, safe anywhere. |
| `logo-favicon.svg` | The mark retuned for 16–24px. Used below about 28px. |
| `README.md` | Modes, keys, the measurement definitions, and the design notes. |
| `.nojekyll` | GitHub Pages marker — present, but Pages is not switched on. |

### The artifact copy

`funtype.html` is generated from `index.html` by stripping the first five lines and the last two:

```bash
sed '1,5d' index.html | sed '$d' | sed '$d' > funtype.html
```

This recipe is **line-count sensitive**, which is a sharp edge worth knowing about. Anything inserted into the first five lines silently changes what gets stripped. When the favicon was added on 2026-09-22 it was deliberately placed on line 7, below the `<title>`, so the strip still cuts exactly the wrapper and nothing else.

### Storage

Everything persistent lives in `localStorage` under the single key **`funtype.v1`**:

```json
{
  "mode": "time",
  "amount": { "time": "30", "words": "25", "text": "short", "code": "js" },
  "punctuation": false,
  "numbers": false,
  "theme": "dusk",
  "bests": { "time-30": { "wpm": 78, "acc": 96, "at": 1758499200000 } },
  "tests": 42,
  "seal": "3kq9zv1x0b2"
}
```

`seal` is a hash of every other field, so a save edited by hand no longer matches and is discarded. Each field is also validated against the values the app can produce before it is used. Both are deterrents against casual tampering, not protection against someone who reads the source — see the README's *Tamper guards*.

Both the read and the write are wrapped in `try`/`catch`. If storage is blocked — private windows, hardened browser settings — the app runs normally for the session and simply forgets afterwards, rather than failing to start. The `v1` in the key is the version handle described in §4.

**Nothing is transmitted.** There is no analytics, no error reporting, no backend. Records stay on one browser on one machine.

### Dependencies

Exactly one external dependency: a Google Fonts stylesheet supplying **Gabarito** (interface) and **JetBrains Mono** (everything you type). Both are preconnected. If the request fails, the CSS falls back to Segoe UI and Cascadia Mono / `ui-monospace` and the app remains fully usable — slightly off-brand, never broken.

The favicon is embedded as a `data:` URI rather than a file reference, so the single-file promise survives even if `index.html` is moved somewhere on its own. The results chart is inline SVG generated at runtime; no charting library.

### Browser support

Any current evergreen browser. The app leans on `localStorage`, CSS custom properties, flexbox and inline SVG — all long-settled. There is no polyfill layer and none is warranted. The layout is desktop-first by nature: this is a physical-keyboard product, and §9 states mobile explicitly out of scope.

## Design system and brand

### Token model

Every colour in the app resolves through a CSS custom property. There are no hardcoded hex values in component rules, which is what makes theming a data change rather than a stylesheet rewrite. A theme is a plain object of eleven values, applied by setting the properties on `:root`.

| Token | Role |
| --- | --- |
| `--bg` / `--surface` / `--raise` | Page, card, elevated card |
| `--text` / `--sub` / `--sub-soft` | Typed text, untyped text, faint text |
| `--accent` / `--accent-deep` / `--accent-ink` | Caret and brand; keycap skirt; the colour that sits *on* accent |
| `--error` / `--error-extra` | Wrong characters; characters typed past a word's end |

### The eight themes

`dusk` (default), `daylight`, `matcha`, `cobalt`, `nordfall`, `bubblegum`, `mono`, `terminal`.

Six are dark and two are light, and the set is deliberately not eight variations on one idea — `bubblegum` is pink-on-cream, `terminal` is phosphor green on near-black, `mono` is greyscale. Because every theme supplies its own `--accent-ink` that contrasts hard against its `--accent`, brand elements recolour correctly in all eight without per-theme overrides.

Chart series are fixed at `#d8792e` (net), `#6189e0` (raw) and `#cf4a66` (errors), checked for colour-blind separation against both the dark and light surfaces. They do not follow the theme, because a chart legend that changes meaning between themes is a chart you cannot read twice.

### Typography

**Gabarito** for the interface — a warm geometric sans that keeps the chrome from feeling clinical. **JetBrains Mono** for everything you actually type, so character widths are uniform and the caret advances predictably. The distinction is load-bearing: a proportional face in the typing area would make the caret jump unevenly and the per-character error marks would not line up.

### The mark

The logo is a **keycap whose legend is a text caret** — the same caret you chase across the screen while typing. It blinks on the same 1.05s beat as the real one, and the cap face depresses 2.6px when pressed, because the mark is also the restart button. It is the rare logo that is a working control.

**Redrawn 2026-09-22.** The original placed the caret inside a dark inset "dish", which left the caret about 10% of the frame. At the 38px it ships at in the header it collapsed into a smudge — the mark read as an orange square with a hole in it. The redraw removes the dish, puts the caret in `--accent-ink` directly on the cap, and grows it to 63% of the cap height as an I-beam: four shapes down to two.

A separate `logo-favicon.svg` handles small sizes, with the cap pushed out to the edges and the caret thickened, because at 16px the 4px skirt and the thin serifs silt up. The swap happens at about 28px.

The wordmark is Gabarito ExtraBold at `-0.022em` tracking, `Fun` in accent and `Type` in text colour. In `logo.svg` it is **live text, not outlined paths** — correct inside the app, which serves Gabarito, but it must be outlined before the file goes to any context that will not have the font.

## Roadmap

Phases are ordered by *ratio of value to simplicity spent*. Each one names what it costs, because the single-file constraint in §5 is the project's main asset and every phase draws on it.

### Phase 0 — Publish it · *recommended, do this first*

Switch on GitHub Pages and give FunType a URL. The `.nojekyll` marker is already committed, so this is a repository setting and a link, not a code change.

**Why first:** everything else in this roadmap is worth more once the thing is reachable, and worth very little while it is a file on one laptop. It is also the only phase that costs nothing architecturally.

**Cost:** none. **Effort:** minutes.

### Phase 1 — Content depth

The word list is 298 entries with 9 duplicates, and there are three text passages and three code snippets. That is enough to prove the modes and thin enough that a regular user will start recognising material, which quietly inflates scores.

- De-duplicate and expand the word list toward ~1,000 entries.
- Add passages, and more code languages — TypeScript, Go, shell.
- Consider a *hard words* toggle drawing from a low-frequency list.

**Why here:** it directly protects the integrity of the measurements in §4, and it is pure data — no new UI, no new state.

**Cost:** file size. A 1,000-word list plus more passages pushes `index.html` past ~60 KB. Still trivially servable, but it is the first real pressure on the one-file rule.

### Phase 2 — History and trend

Today a run is compared only against the single best for its configuration. Keep the last *n* results per configuration and show a trend line, so a user can see a plateau or a regression rather than only a high-water mark.

**Why here:** it is the most-requested capability of any trainer and the storage model already has a natural home for it.

**Cost:** a schema change to `funtype.v1`, which needs a migration path or a version bump. Records must survive it — see §8.

### Phase 3 — Per-character weakness analysis

The engine already knows every keystroke that missed. Aggregating those into a per-character error profile would let FunType say *you lose most of your time on `;`, `p` and capitalised words* — and then generate a drill weighted toward exactly those characters.

**Why here:** this is the genuine differentiator. It converts the measurement work in §4 from a scoreboard into coaching, and no amount of leaderboards substitutes for it.

**Cost:** the largest of the four. New persisted state, a new results panel, and a generator that biases word selection. This is the phase most likely to break the single-file rule, and the one where breaking it would be justified.

### Phase 4 — Shareable results · *optional*

Encode a finished run into a URL fragment so a result can be linked without a server or an account.

**Why last:** pleasant, not load-bearing, and it is the phase most at risk of dragging the product toward the leaderboard shape §2 deliberately rejects.

**Cost:** low technically. The risk is to positioning, not to code.

### Explicitly deferred

Accounts and cloud sync, multiplayer races, a mobile layout, lesson plans or a curriculum, and teacher/cohort tooling. Each contradicts something in §2. They are listed so that deferring them reads as a decision rather than an oversight.

## Risks, constraints and open questions

### Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| **Records are on one machine only** | A user who clears site data, switches browser or replaces a laptop loses every personal best, with no recovery | Accepted — it is the price of having no backend, and §2 treats it as a feature. Mitigate with an export/import of the `funtype.v1` blob, which is far cheaper than accounts |
| **Changing the metrics invalidates every stored best** | Silent corruption: old records compared against new definitions, with no keystroke history to recompute from | Settle §4 before publication. Any later change bumps the key to `funtype.v2` and retires old records honestly rather than misreading them |
| **Content is small enough to memorise** | 298 words with 9 duplicates, 3 passages, 3 snippets — a regular user starts recognising material, which inflates scores and hides real plateaus | Roadmap Phase 1, which is why it sits ahead of the more interesting work |
| **Google Fonts is a single point of failure** | Offline or blocked, the app renders in fallback faces | Already handled: the stack degrades to Segoe UI / Cascadia Mono and stays fully usable. Only the brand suffers |
| **The `sed` regeneration step is line-count sensitive** | Editing the first five lines of `index.html` silently corrupts `funtype.html`, and the corruption is not obvious | Documented in `README.md` and in §5. A more robust marker-based strip would remove the hazard entirely |
| **Single maintainer, no tests** | There is no automated check that a change to the metrics engine did not alter a number | Live with it at this scale, but a small set of fixtures — a known keystroke sequence with known expected outputs — would protect exactly the code that matters most |

### Known wrinkles in the current build

Minor, and none affect correctness of the headline numbers:

- The word list holds **9 duplicate entries** (`door` appears three times; `high`, `small`, `late`, `water`, `build`, `start`, `story` and `watch` twice each). Harmless, but it skews frequency slightly and is untidy.
- `consistency()` opens with a filter that is a no-op — it retains every element and copies the array, nothing more. Dead code rather than a bug.
- The app is desktop-shaped with no mobile layout. Reasonable for a physical-keyboard product, but the page does not currently *say* so on a small screen.

### Open questions

1. **Should consistency be surfaced more prominently?** §4 argues it is the most diagnostic number available, yet it sits in the secondary facts row while accuracy gets headline treatment. That may be the wrong emphasis.
2. **Should a personal best require a minimum accuracy?** Today a best is claimed on net WPM alone. A fast, careless run can set a record that a slower, cleaner run cannot beat — arguably the exact behaviour §2 criticises in other trainers.
3. **How much content is enough** before memorisation stops mattering? Phase 1 proposes ~1,000 words, but that figure is an estimate rather than a measured threshold.
4. **Is the single-file constraint worth keeping through Phase 3?** It should be an explicit decision at that point, not a default that quietly breaks.

## Success measures and delivery

### What good looks like

FunType has no analytics and this proposal does not suggest adding any — which rules out the usual engagement metrics and is a deliberate constraint, not an oversight. The measures below are therefore ones a single maintainer can judge honestly without instrumenting users.

| Measure | Target | How it is judged |
| --- | --- | --- |
| **Time to first keystroke** | Under 2 seconds from opening the URL | Stopwatch. The words must already be on screen; typing is what starts the test |
| **Return use by the maintainer** | Used voluntarily for practice, not just for testing | Honest self-report. A trainer its own author avoids is not finished |
| **Measurement survives scrutiny** | A skeptical typist reads §4 and agrees the numbers are fair | Qualitative, and the one that matters most — the product's whole claim is honest measurement |
| **Runs anywhere, unchanged** | Opens identically from `file://`, a USB stick and a web host | Direct check on each before release |
| **Themes hold up** | All eight legible, brand mark correct in each | Visual pass; already verified for dusk, daylight, bubblegum and terminal |

### Effort

The existing build is complete, so these are incremental estimates for one person working in focused sessions:

| Phase | Estimate |
| --- | --- |
| Phase 0 — publish | Under an hour |
| Phase 1 — content depth | 1–2 sessions, mostly sourcing and de-duplicating |
| Phase 2 — history and trend | 2–3 sessions, including the storage migration |
| Phase 3 — weakness analysis | 5–8 sessions; the only phase that warrants real design work first |
| Phase 4 — shareable results | 1–2 sessions |

No infrastructure cost at any phase. GitHub Pages is free for a public repository, and there is nothing to host beyond static files.

### Out of scope

Stated plainly so it does not have to be re-argued: **accounts and cloud sync, multiplayer racing, a mobile or touch layout, lesson plans and curricula, teacher or cohort tooling, advertising, and any form of telemetry.** Each of these contradicts a position taken in §2. If one is later wanted, it should be adopted as a deliberate change of direction with this section amended — not slipped in as a feature.

### Recommendation

**Do Phase 0 now** — it is an hour of work and every other decision is easier once the thing has a URL. **Settle the two open questions about the metrics** (§8, items 1 and 2) before publishing, because both touch §4 and both get expensive the moment real records exist. **Then do Phase 1**, which protects the measurements, before anything more interesting.

Phase 3 is where the actual product is. Everything before it is getting into position to build it honestly.
