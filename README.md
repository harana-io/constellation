# Constellation ★

A pass-and-play party game for **2–4 players** that runs in any modern mobile browser, fully offline once loaded. One phone, one table, 1–5 rounds, configurable strokes, ~12–22 minutes per session. Designed for bars, dinner parties, and the back booth.

**Live link:** https://harana-io.github.io/constellation/
**Repo:** https://github.com/harana-io/constellation
**Working directory:** `~/Documents/Claude/Projects/Constellation/`

---

## What is this?

Each player is secretly assigned a shape (cat, rocket, ghost, lightning bolt...). The phone passes around the table. On your turn you add **one** brushstroke to your own canvas, trying to evoke your shape without giving it away too obviously. You get a configurable number of strokes (3–6) over the course of a round. After everyone has drawn, the phone passes one more time and each player privately *names* what they think every other player was drawing.

The constraint of a small stroke budget forces stylized, almost cubist art. The private free-text guessing makes the reveal dramatic — nobody knows what others wrote until the truth comes out. Scoring rewards both being read correctly *and* reading others correctly, so you stay engaged on every turn.

---

## The design rationale (so a new session can pick this up cold)

The user asked for: a creative novel mobile browser game; iOS/Chrome compatible; 2–4 players (with optional teams-of-2); ~20 minutes max per game; fun for all ages; bar/dinner setting; pass-and-play *or* Bluetooth multi-device; deployable to Git for instant link sharing; addictive replay value; strong immersive design.

Two important hard constraints we navigated:
1. **iOS Safari has zero support for Web Bluetooth.** True phone-to-phone WebRTC also needs an internet signaling step. So genuinely-offline multi-device play in a mobile browser is essentially impossible. The answer that *feels* right for a bar setting is single-phone pass-and-play: no pairing friction, no four-battery anxiety, everyone watches the reveal together. We leaned into this as a feature, not a workaround.
2. **A single HTML file deployed to GitHub Pages is the lowest-friction sharing pattern.** No build step, no framework, no server. Just `git push` → Pages on → link.

The game concept needed to be novel enough not to feel like "Pictionary with extra steps." The novel mechanics that make Constellation specifically distinct:
- **Strict stroke budget** (3–6 marks) forces highly stylized, abstract art rather than recognizable Pictionary doodles.
- **Simultaneous private voting via free-text** (not multiple choice) — players type what they *think* was drawn, no hint pool. This rewards genuine interpretation and creates very funny mismatches at the reveal.
- **Asymmetric scoring** — you score when others read your art correctly AND when you read others' art correctly, so passive players don't get bored.
- **Round-by-round narrative pacing** — short flavor screens between rounds ("A Fresh Sky," "Mid-Sky," "Deep Sky," "Final Constellation") that scale to whatever round count the table picks.
- **Optional teams mode** at 4 players (2v2) for "dinner couples" energy.

The cosmic visual theme (deep navy gradient, glowing colored ink strokes per player, twinkling starfield, serif italic display type) was chosen to make the game feel like an artifact rather than a utility — a candle on the table, not an app.

---

## Rules

1. **2–4 players** sit around one phone.
2. On the setup screen pick: **Players** (2/3/4), **Rounds** (1–5), **Strokes per player** (3–6), and for 4 players optionally **Teams** (2 vs. 2).
3. Each player is **secretly assigned a shape**. They look at it privately and pass the phone.
4. **Drawing phase.** Round-robin. On your turn you add exactly **one** brushstroke to *your* canvas (lift finger = end of stroke). You get N strokes total per round, the setup setting.
5. **Voting phase.** The phone passes once more. Each player privately *types* what they think each other player was drawing. Votes are sealed — no one sees others' guesses until the reveal.
6. **Reveal.** For each player, their canvas is shown alongside the truth and every voter's guess, with correct guesses highlighted green. Fuzzy matching: minor typos and casing differences count as correct.
7. **Scoring**
   - **+2** for each shape you correctly identify on someone else's canvas.
   - **+1** for each voter who correctly reads your canvas.
   - Teams mode: members of a team share a combined score.
8. After the last round, the **scoreboard** shows the winner (or winning team).

A typical 4-player, 3-round, 4-stroke game runs ~18–22 minutes. A quick 2-player tester runs in under 5.

---

## How to share with friends (it's already deployed)

The game is live at **https://harana-io.github.io/constellation/** — that's the URL to text people. iOS Safari, Chrome on Android, any modern mobile browser. After the first load it works offline (no internet needed at the bar).

**Add to Home Screen recommended:** open the link in Safari → Share icon → "Add to Home Screen". Launches full-screen like a native app, runs offline, no chrome around the canvas.

---

## File structure

```
Projects/Constellation/
├── index.html          # The entire game — single file, no dependencies, no build step
├── README.md           # This file
├── .gitignore          # .DS_Store, node_modules, log, IDE
├── .claude/
│   └── launch.json     # Local preview server config (for the Claude preview tool)
└── .git/
```

`index.html` is self-contained:
- Inline `<style>` for all CSS (no external fonts; Cormorant Garamond + system serif fallback)
- Inline `<script>` for all JS (no framework, vanilla DOM + Canvas 2D)
- Web Audio API for synthesized celestial chimes (no audio files)
- Animated starfield on a fixed background canvas
- Foreground canvas for drawing strokes, scaled to devicePixelRatio

---

## Code architecture (so you can extend it)

**State is one global object `S`.** Every screen reads from and writes to `S`, then calls `render()` which clears `#app` and re-renders the current screen from scratch based on `S.screen`. There's no virtual DOM, no diffing — just full re-renders on every state change. The game has fewer than a dozen screens so this stays fast.

**Screen dispatch:**
```js
function render() {
  const root = app();
  root.innerHTML = '';
  const screen = ({
    title:          renderTitle,
    setup:          renderSetup,
    twist:          renderTwist,
    'reveal-shape': renderRevealShape,
    'pass-draw':    renderPassDraw,
    draw:           renderDraw,
    'pass-vote':    renderPassVote,
    vote:           renderVote,
    'pass-reveal':  renderPassReveal,
    reveal:         renderReveal,
    score:          renderScore,
  })[S?.screen || 'title']();
  screen.classList.add('fade-in');
  root.appendChild(screen);
}
```

**IMPORTANT bug history.** An early version of this object literal referenced an `end: renderEnd` entry where `renderEnd` was never defined. Because JS object literals evaluate every value eagerly at construction time, this threw `ReferenceError: renderEnd is not defined` on *every* render call. The UI was invisible — only the starfield (which runs in its own IIFE before render is called) showed. **If you ever see "just the starfield, no UI": check the dispatch map for undefined references.**

**Key state fields on `S`:**
- `S.players[]` — `{name, color, score, shape, strokesByPlayer:[], receivedVotes:{}, teamIdx}`
- `S.strokes` — strokes-per-player-per-round, set on setup screen (3–6, default 4)
- `S.totalMatches` — rounds-per-game (1–5)
- `S.teamsMode` — boolean, only meaningful at 4 players
- `S.match` — current round index (0-based)
- `S.drawTurnPlayer` / `S.drawTurnIndex` — round-robin draw turn tracking
- `S.voteIndex` / `S.voteTargetIdx` — pass-and-vote pointer pair
- `S.voteSelections` — `{voterName: {targetName: guessString}}`

**Drawing canvas.** Strokes are stored normalized to 0..1 coordinates so the canvas can be replayed at any size (used in reveal). The drawing canvas uses `pointer*` events with `touch-action: none` so iOS Safari doesn't intercept the scroll. The "one stroke per turn" rule is enforced by an `endedTurn` latch — once pointerup fires and the stroke has >1 point, the turn advances after a 350 ms beat.

**Voting is free-text.** Fuzzy match in `tallyVotes()` lowercases, strips punctuation, and accepts substring matches against the shape's name. Strict equality would be too punishing in a party setting.

---

## Local development

```bash
cd ~/Documents/Claude/Projects/Constellation
npx --yes serve . -l 4173
# then open http://localhost:4173 in your browser
```

Or via the Claude preview server: a config exists at `~/Documents/Claude/.claude/launch.json` named `constellation` that serves this directory.

No build, no install, no dependencies. Edit `index.html`, refresh.

---

## Deploying changes

```bash
cd ~/Documents/Claude/Projects/Constellation
git add -A
git commit -m "your message"
git push
```

GitHub Pages auto-rebuilds in ~30–60 seconds. The live link at https://harana-io.github.io/constellation/ updates on the next page load (force refresh / clear cache if your phone has aggressively cached the file).

The GitHub repo is `harana-io/constellation`, owned by the user's `harana-io` GitHub account (authenticated via `gh` CLI).

---

## Resuming work in a new chat

If you're an AI assistant picking this up cold, here's the minimal context:

1. **Project location:** `/Users/earllacey/Documents/Claude/Projects/Constellation/`
2. **What it is:** A pass-and-play party game shipped as a single `index.html` file. Already deployed to GitHub Pages at the live link above. The user (Earl Lacey, `earllacey13@gmail.com`) plays it with friends at bars/dinners.
3. **No build step.** Edit `index.html` directly. Test by serving the folder with any static server. Push to GitHub to deploy.
4. **The single hardest bug** in this codebase's history was the `renderEnd` ReferenceError documented above. If the page renders only the starfield, look there first.
5. **The user prefers pass-and-play over Bluetooth/multi-device.** This was a deliberate design call after we confirmed Web Bluetooth is fully unsupported on iOS Safari. Don't reopen this unless asked.
6. **Adjustable settings on the setup screen:** Players (2–4), Rounds (1–5), Strokes per player (3–6), Mode (Free for All / Teams, 4p only).
7. **All visual + audio polish lives in a single inline `<style>` and a small Web Audio helper at the top of `<script>`.** No external assets.

---

## Ideas / roadmap

These are unbuilt — sketched here so a future session can pick them up.

- **Tie-breaker mini-round** when scores are level — single sudden-death draw with a shared random shape.
- **Sabotage stroke** as a gameplay-active twist: after your final stroke, place one mark on an opponent's canvas. Re-enable the `TWIST[]` array mechanics with real implementations (the original copy promised this but the code never delivered — see commit history).
- **Whisper titles** that each player writes after drawing, shown only at the reveal for narrative flair.
- **Custom shape packs** (NSFW for bars, holiday-themed, sci-fi, food-only).
- **Replay/share image** of the final composition with the answer key, so groups can post their favorite ridiculous drawings.
- **Sound design pass** — current chimes are basic Web Audio sine pulses. A short ambient pad behind the title and reveal would deepen the atmosphere.
- **Genuine multi-device offline** would require WebRTC + LAN-only signaling (mDNS/Bonjour), or one-phone-as-host via Web Bluetooth (Android-only, never iOS). Probably not worth the complexity for a bar game.

---

## Known limitations

- **Web Bluetooth doesn't work on iOS Safari.** Apple platform decision, not fixable in code. Single-phone pass-and-play is the design answer.
- **Free-text voting can have edge cases** when a shape name overlaps with common words. The fuzzy matcher is forgiving by design but not perfect.
- **Drawing on a small phone screen with a finger** is intentionally low-fidelity — part of the game's charm.
- **No persistence.** Refresh the page and the game state is gone. By design for a single-evening party game.

---

## Credits

Made with Claude in a single session for nights out where the conversation slows but no one wants to leave yet. If you find a bug or have an idea, open an issue on the repo.
