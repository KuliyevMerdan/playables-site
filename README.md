# Playable Ads — Portfolio

Fourteen HTML5 playable ads. Each one is a **single HTML file** with no libraries, no external assets and synthesized audio, and each is ready to hand to an ad network as-is.

**Live showcase:** https://kuliyevmerdan.github.io/playables-site/

| | |
|---|---|
| Playables | 14 |
| Runtime dependencies | 0 |
| Asset files (images, audio, fonts) | 0 |
| Size per creative | ~24–59 KB, uncompressed |
| Ad SDK | MRAID-aware, with a `window.open` fallback |
| Target networks | Meta · AppLovin · Unity · ironSource |

## The playables

| Creative | Genre | Tech | Size | What's interesting |
|---|---|---|---|---|
| [Pull the Pin](pull-the-pin/index.html) | Physics puzzle | Canvas 2D | 30 KB | Verlet ball physics with segment collisions from scratch; context-aware tutorial hand |
| [Gem Blast](match-3/index.html) | Match-3 | Canvas 2D | 28 KB | Headless logic core; goal tuned by simulating a weak player so the ad is always winnable; colorblind-safe gem shapes |
| [Sky Tower](stack-3d/index.html) | 3D stacker | Raw WebGL | 24 KB | Hand-written matrix math and lambert shading, no three.js; a 2D effects layer projected over the 3D scene |
| [Stride Studio](sneaker-studio/index.html) | Brand / e-commerce | Canvas vector + DOM | 27 KB | Sneaker configurator; the end card shows the colorway the player built, with price and an order CTA |
| [Unread](chat-story/index.html) | Chat story | DOM | 28 KB | Branching script engine with timed choices; picks for the player on timeout so the ad always finishes |
| [Bolt Out](bolt-out/index.html) | Screw puzzle | Canvas 2D | 35 KB | Slot management plus plate layering; a free bonus slot appears when the player is stuck, so there is no dead end |
| [Crowd Rush](crowd-rush/index.html) | 3D runner | Raw WebGL | 42 KB | The whole crowd is three instanced draw calls; ×2 / ÷2 gates, saws, pit bridge, brick wall finale |
| [Jam Out](jam-out/index.html) | Parking jam | Canvas 2D | 38 KB | Puzzle solved as a static dependency graph; the last lot ends in a real deadlock that a crane resolves |
| [Bubble Pop](bubble-pop/index.html) | Bubble shooter | Canvas 2D | 47 KB | Hex grid; the aim line predicts wall bounces and shows the exact landing cell |
| [Merge Bakery](merge-bakery/index.html) | Merge-2 | Canvas 2D | 46 KB | Chain merges and a generator that levels up, which fits an exponential economy into a ~23-second ad |
| [Word Harbor](word-harbor/index.html) | Word swipe | Canvas 2D | 42 KB | No dictionary is shipped: every word is an anagram of the level's letters, so all word data is 963 bytes. English and Russian |
| [Tap Eats](tap-eats/index.html) | Brand / app flow | Canvas UI | 46 KB | A delivery-app flow in three taps; the end card is a receipt for the order the player built |
| [Demolition Row](demolition-row/index.html) | Rigid-body physics | Custom 2D engine | 48 KB | Rigid-body solver written from scratch (OBBs, SAT contacts, split impulses, sleeping). It is deterministic, so every pair of charge positions is simulated to rest as a test |
| [Butcher Hero](butcher-hero/index.html) | Action RPG | Canvas 2D | 59 KB | Eat what you kill for stat gains; 7 waves and a boss in a 3–5 minute session that can be lost; balance measured with autopilot and 20-seed stress runs |

Sizes are the raw `index.html` byte counts. Every creative also has a WebAudio synth for SFX, plus music in Butcher Hero.

## Repository layout

```
.
├── index.html          # Entry point: redirects to showcase/
├── showcase/index.html # Portfolio page; embeds each playable live in a phone frame (iframe)
├── portfolio.html      # Offline portfolio: the same page with all 14 creatives base64-inlined
├── <creative>/index.html  # One self-contained playable per folder
└── .nojekyll           # Tells GitHub Pages to serve the files as-is
```

- **`showcase/`** loads the creatives from their folders over relative URLs, so it needs an HTTP server.
- **`portfolio.html`** (~750 KB) works anywhere, including opened straight from disk or sent as an email attachment. It mounts each creative through `srcdoc` only when the card scrolls into view, and "Open ⤢" opens it in a new tab through a Blob URL.

## Running locally

No build step. Serve the repo root:

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000/. Any single creative also runs when opened directly from disk.

## How each creative is built

The same structure is used in every folder:

- **Virtual resolution + letterboxing.** The scene is laid out in a fixed portrait space (e.g. 720×1280) and scaled to any viewport. Safe-area insets are respected.
- **`CONFIG` block at the top of the script.** It holds the store URL and all tuning values (timings, difficulty, `autoEnd` to force the end card).
- **MRAID-aware start.** If `window.mraid` exists, the game waits for `ready`. Otherwise it starts immediately.
- **CTA.** `openStore()` calls `mraid.open(storeURL)` and falls back to `window.open`. When a creative runs inside the showcase (its iframe `name` starts with `pa-demo`), the CTA shows a demo notice instead of leaving the page.
- **Audio.** SFX are synthesized with WebAudio oscillators, and the context is resumed on the first user gesture. Every creative has a mute button.
- **Haptics.** `navigator.vibrate` is used where available.
- **A game that always finishes.** Idle hints, rescue mechanics and a forced end card by timeout mean the ad reaches the CTA whatever the player does. Butcher Hero is the one exception, since it is losable on purpose.

### Preparing a creative for a campaign

1. Copy `<creative>/index.html`.
2. Set `CONFIG.storeURL` (currently the placeholder `https://apps.apple.com/app/id000000000`).
3. Optionally tweak the `CONFIG` tuning values.
4. Upload the single file to the network. If the network needs a different CTA call, adapt `openStore()`.

## Testing hooks

Game logic runs on **simulation time** (`S.time`), not wall-clock time, so a whole session can be stepped without a browser event loop. Each creative exposes a debug handle:

```js
window.__game.tick(600);   // advance 600 frames at 1/60 s
window.__game.S            // live game state
window.__game.CONFIG       // tuning values
window.__game.end();       // jump to the end card
```

The exact helpers differ per game. For example, Bolt Out also exposes `tap(id)`, `tapAt(x, y)`, `auto()` and `hint()`. These hooks drive automated flow tests, balance simulations and the deterministic physics checks mentioned above.

## Deployment

The site is deployed with GitHub Pages from `main` (repo root). The root `index.html` redirects to `showcase/`, so the bare URL works.

## Contact

Merdan Kuliyev — merdankuliyev1402@gmail.com
