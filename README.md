# 🪙 FlipaCoin

> flipping a big fat coin

A zero-dependency, single-file coin flip web app with animations, sound, stats tracking, and persistent history.

---

## 🚀 How to Run

No installs. No build step. No server required.

Just open `index.html` in any browser — double-click it in Finder/File Explorer, or use VS Code's Live Server extension for auto-reload while editing.

---

## 📁 File Structure

```
FlipaCoin/
├── index.html   ← the entire app (HTML + CSS + JS in one file)
└── README.md
```

Everything lives inside `index.html`. There are no external files, no npm packages, no frameworks.

---

## 🏗️ Architecture Overview

The file is split into three sections inside one HTML document:

```
index.html
├── <head>         → fonts + all CSS styles
├── <body>         → HTML structure (the UI)
└── <script>       → all JavaScript logic
```

---

## 🎨 CSS — How It's Styled

### Design Tokens (CSS Variables)
At the top of the `<style>` block, a `:root` block defines every color used in the app:

```css
:root {
  --bg:           #0d0d12;      /* near-black page background */
  --surface:      #16161f;      /* card backgrounds */
  --gold-light:   #f9d96b;      /* heads color (bright) */
  --gold-mid:     #c89c2a;      /* heads color (mid) */
  --gold-dark:    #7a5c10;      /* heads color (dark) */
  --silver-light: #e8eaf0;      /* tails color (bright) */
  --silver-mid:   #a0a8b8;      /* tails color (mid) */
  --silver-dark:  #4a5068;      /* tails color (dark) */
  --accent:       #7c6bff;      /* purple — button, total counter */
  --accent2:      #ff6b9d;      /* pink — button gradient end */
}
```

This makes it easy to retheme the whole app by changing just these values.

### Fonts
Two fonts are imported from Google Fonts:
- **Space Grotesk** — used for all body text, labels, and buttons
- **Space Mono** — monospace font used for numbers and the result banner (HEADS / TAILS)

### Key CSS Sections

| Section | What it does |
|---|---|
| `#stars` / `.star` | Positions 80 tiny dots fixed behind everything, animated with `@keyframes twinkle` |
| `.coin-stage` | A `perspective: 800px` container that creates the 3D depth effect |
| `.coin` | The flippable element using `transform-style: preserve-3d` |
| `.coin-face` | Each face (heads/tails) uses `backface-visibility: hidden` so only one shows at a time |
| `@keyframes coinSpin` | Rotates the coin 3600° (10 full rotations) over 1.4 seconds |
| `.coin.land-heads/tails` | Snaps the coin to the final position with a springy `cubic-bezier` bounce |
| `.coin-stage::after` | An invisible overlay that becomes a gold or silver glow ring after landing |
| `.stat-value.pop` | A quick scale-up animation that plays on the counter that just changed |
| `.pill` | The small circular history icons, each animated in with a spin-scale entrance |

---

## 🧱 HTML Structure

The visible UI is built inside a `.wrapper` div, stacked vertically:

```
<body>
├── #stars                  ← animated starfield (fixed, behind everything)
├── #confetti-canvas        ← fullscreen canvas for confetti particles
├── .sound-toggle           ← 🔊 button fixed to top-right corner
└── .wrapper                ← centered column, max-width 620px
    ├── <header>            ← title + subtitle
    ├── .coin-stage         ← 3D coin container
    │   └── .coin           ← the coin itself
    │       ├── .coin-heads ← front face (gold, 👑)
    │       ├── .coin-tails ← back face (silver, 🦅), pre-rotated 180°
    │       └── .coin-edge  ← border ring that changes color on result
    ├── .result-banner      ← "HEADS" or "TAILS" text, fades in after flip
    ├── .flip-btn           ← the main button
    ├── .stats-grid         ← 3-column grid: Heads count, Tails count, Total
    ├── .info-row           ← Win Rate % | Current Streak | Longest Streak
    ├── .bar-section        ← heads vs tails ratio progress bar
    ├── .history-section    ← last 30 flips shown as mini coin pills
    └── .reset-btn          ← clears all stats (with confirmation)
```

---

## ⚙️ JavaScript — How the Logic Works

### State Object
All data lives in one plain object:

```js
state = {
  heads: 0,               // total heads count
  tails: 0,               // total tails count
  total: 0,               // total flips
  history: [],            // array of 'heads'/'tails', newest first, max 30
  currentStreak: 0,       // how many in a row right now
  currentStreakType: null, // 'heads' or 'tails'
  longestStreak: 0,       // all-time record streak
}
```

This state is saved to `localStorage` after every flip, so it survives page refreshes.

### Core Functions

#### `flip()`
The main function. Called when the button is clicked, or Space/Enter is pressed.

1. Sets `isFlipping = true` and disables the button (prevents double-flips)
2. Calls `Math.random() < 0.5` to determine heads or tails
3. Plays the whoosh sound via `playWhoosh()`
4. Adds `.spinning` class to the coin → triggers the CSS spin animation
5. After 1400ms (when the spin ends), removes `.spinning` and adds `.land-heads` or `.land-tails`
6. Plays the landing clink sound via `playLand()`
7. Shows the result text and glow ring
8. Calls `updateState()` and `launchConfetti()`
9. Re-enables the button

#### `updateState(isHeads)`
Updates the state object after a flip:
- Increments the right counter and total
- Prepends the result to `history[]`, trims it to 30
- Updates current streak (resets to 1 if the type changed)
- Updates longest streak if current beats it
- Calls `save()` to write to localStorage
- Calls `renderUI()` to update the DOM

#### `renderUI(lastType)`
Reads from `state` and updates every visible element:
- Updates the three counter numbers (with a pop animation on the one that changed)
- Updates win rate %, streak display, longest streak
- Animates the progress bar width
- Highlights the winning stat card with a colored border
- Rebuilds the history pills row from scratch

#### Sound — Web Audio API
No audio files are used. All sound is generated programmatically:

- **`playWhoosh()`** — creates a buffer of white noise, runs it through a bandpass filter that sweeps from 800Hz down to 200Hz, fades out over 0.5s. Sounds like a spinning whoosh.
- **`playLand(isHeads)`** — fires 3 sine wave oscillators in quick succession (offset by 30ms each), at slightly different frequencies depending on heads (higher pitch) or tails (lower pitch). Together they sound like a metallic clink.

The `AudioContext` is only created on first use (after a user interaction), which is required by browsers.

#### Confetti — Canvas API
A `<canvas>` element sits fixed over the full page at z-index 50.

- **`launchConfetti(isHeads)`** — pushes 80 particle objects into an array. Each has a random position, velocity, rotation, size, and color (gold palette for heads, silver for tails).
- **`animateConfetti()`** — runs a `requestAnimationFrame` loop. Each frame it clears the canvas, updates every particle's position/rotation/opacity, draws it as a rotated rectangle, and removes dead particles. Stops looping when the array is empty.

#### Starfield
On page load, 80 `<div class="star">` elements are created and injected into `#stars`. Each gets a random size (1–3px), random position, and random CSS custom property values (`--dur`, `--delay`) that control its individual twinkle animation timing.

#### Persistence
```js
function save() {
  localStorage.setItem('coinflip_state', JSON.stringify(state));
}
function load() {
  return JSON.parse(localStorage.getItem('coinflip_state'));
}
```
Called on every flip. On page load, `load()` runs first — if saved data exists, it restores the state and `renderUI()` populates the counters immediately.

---

## ⌨️ Controls

| Action | How |
|---|---|
| Flip the coin | Click the **Flip the Coin** button |
| Flip with keyboard | Press `Space` or `Enter` |
| Toggle sound | Click the 🔊 button (top-right) |
| Reset all stats | Click **Reset Stats** → confirm |

---

## 🌐 Browser Compatibility

Works in all modern browsers (Chrome, Firefox, Safari, Edge). No polyfills needed.

- CSS 3D transforms — supported everywhere
- Web Audio API — supported everywhere (requires a user interaction before first sound)
- Canvas API — supported everywhere
- localStorage — supported everywhere
