# 🪙 FlipaCoin
> *it's giving skill. it's not.*

A single-file coin flip survival game with TikTok live energy, a fake skill system, dummy authentication, and enough psychological manipulation to keep you playing for way too long.

No frameworks. No build step. No dependencies. Just open `index.html` in a browser.

---

## 🚀 How to Run

**Option 1 — Double-click** `index.html` in Finder/File Explorer. Opens in any browser instantly.

**Option 2 — VS Code Live Server** (best for editing):
1. Install the **Live Server** extension in VS Code
2. Right-click `index.html` → **Open with Live Server**
3. Auto-reloads on every save

---

## 📁 File Structure

```
FlipaCoin/
├── index.html   ← the entire app (HTML + CSS + JS, ~1100 lines)
└── README.md
```

Everything is in one file. No `node_modules`, no bundler, no build command.

---

## 🎮 How to Play

1. **Sign up or log in** on the auth screen
2. **Pick Heads or Tails** before every flip (or use `H` / `T` on keyboard)
3. **Flip the coin** — button or `Space`
4. ✅ Correct = streak goes up, you keep going
5. ❌ Wrong = **instant game over**, no second chances
6. 💰 **Cash Out** at streak 3+ to bank your score safely and restart clean
7. 🌡️ **Watch the Heat Meter** — it's a "hint" (it's fake, but your brain won't care)

---

## ✨ Features

### 🔐 Authentication
- **Sign Up** — username (3+ chars, alphanumeric + underscores), email, password (6+ chars), confirm password
- **Log In** — validates against localStorage "database", inline error messages with red input highlighting
- **Social login** — Google, Discord, X/Twitter buttons fake a 1.2s connection delay then auto-create a guest account
- **Persistent session** — saved to `localStorage`, survives page refresh (no re-login needed)
- **@username chip** in the top bar — click to log out
- **Welcome toast** on login — personalized "welcome back" vs "welcome to the coin"

### 🪙 Core Game Loop
- One life. Pick correctly = streak up. Wrong = GAME OVER. No mercy.
- Coin spins faster every 5 correct guesses (spin duration shrinks from 1.1s down to 0.45s minimum)
- **DANGER ZONE** kicks in at streak 5 — flashing warning, bass thump, coin wobbles
- **💰 Cash Out** button appears at streak 3 — save your streak and restart, or risk it all
- **Near miss** — 15% of losing flips show a red glow on the coin right before it lands wrong, making it feel like you *almost* made it

### 🧠 The Skill Illusion (why it's addictive)
- **Heat Meter** — shows which side is "hot." Updates after every flip biased toward the *opposite* of the last result (gambler's fallacy bait). Feels like real information. Isn't.
- **Coin Lean** — coin physically tilts before you flip, suggesting a "tell." Loosely correlated with the heat meter, mostly random. Players think they found a pattern.
- **Streak Multiplier badge** — your streak IS the multiplier number. Going from 5× to 6× triggers a reward loop.
- **Near miss psychology** — losing by "almost" surviving pulls you back in faster than a clean loss

### 💬 Live TikTok Chat (fake)
30 fake accounts (`@skill_issue_lol`, `@ratio_machine`, `@delulu_but_right`, `@chronically_online__`, etc.) spam the right sidebar throughout gameplay. Messages fire from 3 pools:
- **Hype** — "LETS GOOOOO 🔥", "ate and LEFT no crumbs 💅", "bro can predict the future??"
- **Roast** — "cooked. deleted. blocked.", "npc behavior detected", "this is painful to witness"
- **Neutral** — "the heat meter is RIGHT THERE 👀", "i'm nervous for them", "don't mess this up"

Chat reacts contextually — milestones trigger hype waves, game over triggers roast waves, cash out triggers its own comments.

### 😈 Taunts & Roasts
- **Win taunts** — "W rizz on a coin fr", "the universe said yes king", "heat meter never lies 🔥"
- **Lose taunts** — "L + ratio + wrong + skill issue", "should've cashed out bestie 😭", "ratio'd by a literal coin"
- **Game over roasts** — tuned to your exact score. Dying at 1 = "bro lasted one flip 💀 delete the app". Dying at 50 = "50 STREAK AND YOU STILL LOST??"
- **Milestone callouts** — special messages at 5, 10, 15, 20, 25, 30, 40, 50 correct

### 🎨 Visual Effects
- **Animated starfield** — 80 twinkling stars fixed behind everything
- **Background pulse** — subtle color shifts gold/silver based on coin result
- **Confetti burst** — gold palette on win, red on loss, fires on correct guess and cashout
- **Floating emoji reactions** — 🔥💯👑 on wins, 💀😭🪦 on losses, spawn from random positions
- **Screen shake** — wrong guess shakes the whole page
- **Screen flash** — green tint on correct, red on wrong
- **Glow rings** — gold or silver halo around the coin after landing
- **Level-up style toast** — appears on milestone streaks

### 🔊 Sound (Web Audio API — no files needed)
All sound generated in-browser, no audio files:

| Sound | When |
|---|---|
| Whoosh | During coin spin |
| Metallic clink (3 oscillators) | On landing |
| Victory chime (ascending 3 notes) | Correct guess |
| Descending sawtooth buzz | Wrong guess / game over |
| 5-note fanfare | Milestone streaks |
| 4-note ascending chord | Cash out |
| Low bass thump | Danger zone flip |

### 📊 Stats Tracking
- Current streak + session best
- All-time best (persisted in `localStorage` forever)
- Total flips + accuracy % (shown on game over screen)
- Whether you cashed out this run and at what streak

---

## 🏗️ Architecture

The file has three sections inside one HTML document:

```
index.html
├── <head>          → Google Fonts import + all CSS (~310 lines)
├── <body>          → HTML structure for all screens
└── <script>        → all JavaScript logic (~600 lines)
```

### Screens (all `position:fixed`, toggled via `.show` class)
```
Auth Screen          → login/signup, gates everything
  ↓ (on login)
Start Screen         → rules + "i'm built different" button
  ↓ (on start)
Game (live)          → coin, picks, heat meter, chat, cashout
  ↓ (on wrong guess)
Game Over Screen     → score, roast, stats, replay
  ↓ (on cashout)
Cashout Screen       → celebration, saved streak, run it back
```

### Key JS Functions

| Function | What it does |
|---|---|
| `checkSession()` | Runs on load — auto-logs in if valid session in localStorage |
| `doLogin()` | Validates inputs, checks against localStorage "DB", calls `loginSuccess()` |
| `doSignup()` | Validates all fields, hashes password with `btoa()`, saves to localStorage |
| `dummySocial()` | Fakes OAuth delay, auto-creates guest account, calls `loginSuccess()` |
| `loginSuccess()` | Saves session, shows welcome toast, reveals start screen |
| `doLogout()` | Clears session, resets game state, shows auth screen |
| `pick(side)` | Sets prediction, updates button styles, triggers coin lean |
| `doFlip()` | Main game loop — spin animation, outcome, sound, scoring, near miss |
| `updateHeat(result)` | Updates fake heat meter with gambler's-fallacy-biased drift |
| `showLean(side)` | Tilts coin pre-flip as a fake "tell" |
| `doCashout()` | Banks streak, shows cashout screen, resets for next run |
| `showGameOver()` | Shows score, picks roast based on streak, displays stats |
| `restartGame()` | Full state reset, back to live game |
| `addChat(msg, type)` | Injects a fake chat message into the sidebar |
| `burst(win)` | Launches confetti particles (gold=win, red=loss) |
| `floatEmoji(e, x, y)` | Spawns a floating emoji that drifts upward and fades |

### Auth "Database"
Stored in `localStorage` as two keys:
```js
// fca_users — object of all accounts
{
  "adityajjjj": { password: "base64encoded", email: "a@b.com", joined: 1234567890 },
  "google_user_4821": { password: "base64encoded", email: "", social: "Google" }
}

// fca_session — just the username string of whoever's logged in
"adityajjjj"
```
Passwords are encoded with `btoa()` — not real security, just enough to look right. This is a frontend-only dummy system.

---

## ⌨️ Controls

| Key | Action |
|---|---|
| `H` | Pick Heads |
| `T` | Pick Tails |
| `Space` | Flip the coin |
| `C` | Cash out (if available) |

---

## 🌐 Browser Compatibility

Works in all modern browsers. Requires a user interaction before audio plays (browser rule, not a bug). No polyfills needed.
