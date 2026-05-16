# Constellation ★

A pass-and-play party game for 2–4 players. One phone, one table, three matches, about 18 minutes. Works fully offline once loaded.

## The pitch

Each player gets a secret shape. The phone passes around the table. On your turn you add **one** brushstroke to your own canvas, trying to evoke your shape — you get three strokes total. After everyone draws, the phone passes once more and each player privately guesses what every *other* player was drawing.

The constraint of three strokes forces stylized, abstract art. The private voting makes the reveal dramatic. The asymmetric scoring (you earn points for being read correctly *and* for reading others correctly) keeps everyone engaged on every turn.

## Rules

- **2–4 players.** Pass the phone between turns. Only the active player should see the screen.
- **3 strokes per player per match.** Lift your finger to end your stroke — that's your turn.
- **Voting is private.** Hand the phone to each player in turn; their picks are sealed in.
- **Scoring**
  - +2 for each shape you correctly identify on someone else's canvas
  - +1 for each voter who correctly reads your canvas
- **3 matches per session.** Highest total wins the night.

## Quick deploy to GitHub Pages (≈2 min)

1. Make a new public repo on GitHub (e.g. `constellation`).
2. From this folder:
   ```bash
   git remote add origin git@github.com:YOUR_USERNAME/constellation.git
   git branch -M main
   git push -u origin main
   ```
3. On the repo page, **Settings → Pages → Source: `Deploy from a branch` → Branch: `main` / root → Save**.
4. Wait ~30 seconds. Your share link will be `https://YOUR_USERNAME.github.io/constellation/`.

Send that link to your friends. iOS Safari, Chrome on Android, any modern browser — works everywhere. Once it's loaded the first time, it works offline (no internet needed at the bar).

## Why pass-and-play, not Bluetooth?

iOS Safari doesn't support Web Bluetooth at all, and true phone-to-phone WebRTC needs an internet signaling step to handshake. For a bar/dinner game, passing one phone around the table is actually faster: no pairing, no logins, no "wait my battery's dead." Single source of truth, dramatic reveals, everyone watches together.

## Add to home screen (recommended)

On iOS: open the link in Safari → tap Share → **Add to Home Screen**. The game launches full-screen like a native app and runs offline.

## Roadmap / ideas

- Team mode for 4 players (pairs share a score)
- Sabotage stroke (place one mark on an opponent's canvas)
- Custom shape packs (NSFW, holidays, sci-fi)
- Match-3 twist: "Whispers" — each player writes a 1-word title revealed at scoring

## Credits

Made for nights out where the conversation slows but no one wants to leave yet.
