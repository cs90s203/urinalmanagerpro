# 🚽 Urinal Manager Pro

A fast-paced, single-file HTML5 management game where you run a public restroom under increasing chaos. Assign customers to urinals, keep them clean, manage the queue, and survive escalating waves of... very impatient people.

**[▶ Play now](https://s90s203.github.io/urinalmanagerpro/)**

## Gameplay

Customers arrive and join the queue. Drag them onto an open urinal before they lose patience and leave. Urinals get dirty with use — scrub them clean or they'll break. Spend your earnings on shop items (cleaning stickers, air fresheners, extra urinals, time-buying items) to keep up with the growing crowd.

As your score climbs, the game escalates: faster spawns, special customer types with their own quirks, surprise inspections, and — eventually — total chaos.

### Customers

| Type | Behavior |
|---|---|
| 🧍 Regular | The baseline customer |
| 👴 Elder | Slows down neighboring urinals |
| 🧔 Frequent | Comes back often, needs care |
| 🏃 Urgent | High mess, high reward |
| 🥴 Drunk | Long visit, high score |
| 🧑‍🎤 Delinquents | Arrive in noisy groups during peak waves |
| 💂 Soldiers | Large, fast-moving waves — the real test |
| 🕴️ Gangster | Scares off whoever's next to them |
| 🧙 Inspector | Surprise inspection — keep things clean or pay the price |

### Systems

- **Shop**: cleaning items, air fresheners, extra urinal slots, wait-time boosts, and a "bullet time" power-up for precise control during chaos
- **Achievements**: local achievement tracking, a character codex, and a global leaderboard (Firebase-backed)
- **Save system**: auto-saves mid-run so a dropped connection or backgrounded tab doesn't cost you progress
- **Config dashboard**: an in-game panel for tuning game constants — handy for players who want to explore how the balance works

## Tech

Single self-contained HTML file — no build step, no dependencies to install.

- Vanilla JavaScript, inline CSS
- Web Audio API for all sound effects and background music (synthesized, no audio files)
- Firebase Realtime Database for the global leaderboard
- localStorage for save data, achievements, and settings

Just open the `.html` file in a browser, or serve it with any static file server.

## Running locally

```bash
python3 -m http.server 8000
# open http://localhost:8000/index.html
```

## License

[CC BY-NC-ND 4.0](LICENSE) — you're welcome to play it, share a link to it, and read the source for reference. Please don't redistribute modified copies or use it commercially. See [LICENSE](LICENSE) for details.
