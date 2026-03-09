# CLAUDE.md — Super Animal Smash

## Project Overview

**Super Animal Smash** is a browser-based multiplayer fighting game inspired by Super Smash Bros. It features 50+ playable animal characters, local and online multiplayer, multiple game modes, and rich customization options. The entire application is a **single self-contained HTML file** (`animalsmash.html`, ~739 KB, ~12,600 lines).

There is no build system, no package manager, and no external server. Open the file in a browser to play.

---

## Repository Structure

```
animalsmash/
├── animalsmash.html     # The entire application (HTML + CSS + JS)
└── CLAUDE.md            # This file
```

---

## File Architecture

`animalsmash.html` is divided into three major sections:

### 1. HTML + CSS (lines 1–3276)
- `<head>`: External CDN scripts (PeerJS for WebRTC), Google Fonts
- `<style>`: ~1,200 lines of embedded CSS
  - CSS custom properties (`:root`) for the design system
  - Glass-morphism UI theme with `backdrop-filter`
  - `@keyframes` animations for menus, transitions, and effects
  - Mobile control overlays
- `<body>`: Static HTML containers (`#game-container`, `#gameCanvas`, overlay divs for menus, HUD, modals)

### 2. JavaScript — Online Layer (lines 3277–4218)
```
// ========================  PEERJS ONLINE LAYER  ========================
```
WebRTC peer-to-peer multiplayer via PeerJS v1.5.4:
- **Host**: runs authoritative game simulation, broadcasts full state snapshots each frame
- **Guest**: sends only inputs to host each frame, renders received state (thin client)
- Key functions: `serializeState()`, `applyState()`, `initPeer()`, `joinPeer()`
- Lobby UI: character selection for up to 4 players, team assignments, match rules

### 3. JavaScript — Game Engine (lines 4219–12608)
```
// ==================  ORIGINAL GAME CODE  ==================
```

**Global state variables** (lines 4222–4260):
- `gameState` — `'MENU' | 'GAME' | 'GAMEOVER'`
- `players`, `items`, `projectiles`, `bouncyProjectiles`, `decoys`, `puddles`
- `matchRules` — mode, item rate, damage multiplier, gravity, speed, etc.
- `slots` — player/CPU slot configuration
- `AVAILABLE_CHARS` — array of all playable emoji keys

**Major sections within the game engine:**

| Lines | Section |
|-------|---------|
| 4261–4345 | Image character system (`CHAR_IMAGES`, image cache) |
| 4346–4893 | `CHAR_STATS` — core character definitions |
| 4894–5860 | `CHAR_STATS` additions (5 newer characters via `Object.assign`) |
| 5861–6548 | `MAPS` — all stage definitions |
| 6549–6604 | Player control key bindings |
| 6605–8173 | Local play UI: fighter select, stage select, rules, arcade/survival modes |
| 8174–9226 | Game features: replay system, kill cam, pause, mirror match, stage morph, rematch, settings, graphics quality, controls rebinding, cheat codes |
| 9227–9601 | `startGame()`, HUD building, game-over logic |
| 9602–10415 | `class Player` — core player logic |
| 10416–10445 | `class ItemDrop` |
| 10446–10530 | `class Projectile` |
| 10531–10638 | `class Decoy` |
| 10639–10733 | `class BouncyProjectile` |
| 10734–10823 | Particle pool system |
| 10824–11196 | Combat helpers: hit stun, blood splats, `_humanKill`, combo counter |
| 11197–11224 | `spawnItem()`, `spawnItemAt()`, `triggerSuddenDeath()` |
| 11225–11280 | `initGame()`, `updateUI()` |
| 11281–11941 | World rendering: `drawWorldEffects()`, `drawGameWorld()` |
| 11942–12145 | `gameLoop()` — main `requestAnimationFrame` loop |
| 12146–12409 | `showMenuScreen()`, `triggerKOEffect()` |
| 12410–12608 | Item pick-up manager (IPM) UI |

---

## Key Data Structures

### `CHAR_STATS` object
Each key is an emoji string. Each value has:
```js
{
  name: string,           // display name
  desc: string,           // short description
  passive: string,        // passive ability description
  weight: number,         // affects knockback received
  gravity: number,        // gravity multiplier
  accel: number,          // ground/air acceleration
  dmg: number,            // base attack damage multiplier
  kb: number,             // base knockback multiplier
  ranged: boolean,        // true if primary attack is ranged
  onFrame(p, attackObj),  // called each frame during attack animation
  onHit(attacker, target, damage),
  onHitMod(attacker, target, dmg, kb, hX, hW) → {dmg, kb},
  onAttacked(target, attacker),
  overrideAttack(p, x, y, aimed, target) → boolean,
  afterAttack(p),
  ult(p),                 // ultimate ability function
  ultDesc: string,        // ultimate description
}
```

### `MAPS` array
Each map entry:
```js
{
  name: string,
  emoji: string,
  bg: string,             // canvas background color/gradient
  platforms: [{x, y, w, h, color}],
  hazards: [...],         // optional
  music: string,          // optional theme hint
}
```

### `matchRules` object
```js
{
  mode: 'stock' | 'time' | 'coin',
  value: number,          // lives / seconds / coins
  itemRate: number,       // 0 = off, 1 = normal, 2 = high
  ultsOn: boolean,
  dmgMult: number,
  startPct: number,       // starting damage %
  friendlyFire: boolean,
  gravityMult: number,
  gameSpeed: number,
  globalSize: number,
}
```

### `Player` class (line 9602)
Key properties: `x, y, vx, vy`, `percent` (damage %), `lives`, `score`, `dirX`, `isAttacking`, `attackFrames`, `shieldHP`, `hasUlt`, `invincible`, `weapon`, and many passive state fields prefixed with `_` (e.g., `_momStacks`, `_frenzy`, `_wasDashing`).

---

## Adding a New Character

1. Add the emoji to `AVAILABLE_CHARS` array (line ~4259)
2. Add a stats entry to `CHAR_STATS` (around line 4346 or via `Object.assign` at line 4895)
3. Optionally add an image URL to `CHAR_IMAGES` for a custom sprite
4. Optionally add an unlock condition in the survival reward config

Character stats callbacks (`onFrame`, `onHit`, `ult`, etc.) are plain JS functions that manipulate the passed player object directly.

---

## Adding a New Map

Add an entry to the `MAPS` array (line 5861). The platforms array uses `{x, y, w, h}` in game-world coordinates (canvas is 1200×800 logical pixels, y increases downward).

---

## Development Workflow

**No build step required.** Edit `animalsmash.html` directly and refresh in a browser.

- **Local testing**: Open `animalsmash.html` in any modern browser (Chrome, Firefox, Edge)
- **Online testing**: Serve via any static HTTP server (e.g., `python3 -m http.server`) then share the URL; PeerJS handles the WebRTC signaling via their public cloud server
- **Debugging**: Use browser DevTools console; the game loop runs at 60 FPS via `requestAnimationFrame`
- **No linting or formatting tools** are configured

### Graphics/Performance Settings
Quality presets are stored in `localStorage` under the key `gfxSettings`. The `gfx` object controls: particles, shadows, bloom, screen shake, blood effects, and post-processing. Call `applyPreset('low' | 'medium' | 'high' | 'ultra')` to switch.

### Cheat Codes / Unlocks
Unlock logic is in `submitCode()` (line ~9095). Unlocked characters are persisted in `localStorage`.

---

## Code Conventions

| Convention | Details |
|------------|---------|
| Naming | `camelCase` for variables/functions, `ALL_CAPS` for constants |
| Private fields | Underscore prefix: `_wasDashing`, `_momStacks` |
| Loop variables | Single-letter: `i`, `p`, `t`, `a` |
| Section headers | `// ═══ TITLE ═══` (major) or `// ── Title ──` (minor) |
| Indentation | 4 spaces |
| Semicolons | Used consistently |
| Type system | None — vanilla JavaScript only |
| CSS variables | Defined in `:root`, prefixed `--ui-` |

---

## External Dependencies

| Library | Version | Load Method |
|---------|---------|-------------|
| PeerJS | 1.5.4 (fallback: 1.5.2) | CDN script tag |
| Google Fonts (Outfit, Poppins) | latest | CSS `@import` |

Both are loaded from CDN at runtime. The game includes a fallback CDN for PeerJS:
```html
<script src="https://cdn.jsdelivr.net/npm/peerjs@1.5.4/dist/peerjs.min.js"></script>
<script>if(typeof Peer==='undefined'){document.write('<script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"><\/script>');}</script>
```

---

## Game Modes

| Mode | Description |
|------|-------------|
| Local Play | 1–8 players (human + CPU) on same device |
| Online Play | Up to 4 players via WebRTC (P2P, host-authoritative) |
| Arcade | Single-player bracket vs. CPU opponents |
| Survival | Endless waves; score-based character unlocks |
| Mirror Match | All players use the same randomly chosen character |

---

## Online Multiplayer Architecture

- **Host** runs the full simulation at 60 FPS and serializes the entire game state (all player positions, velocities, attack frames, status effects, projectiles) on each frame via `serializeState()`
- **Guest** sends only its button inputs via `guestInputSnapshot` each frame
- State is transmitted as JSON over PeerJS (WebRTC data channel)
- No rollback netcode — designed for low-latency connections (LAN or same network)
- `onlineRole` is `'host'`, `'guest'`, or `null`

---

## Persistence (localStorage)

| Key | Contents |
|-----|---------|
| `gfxSettings` | Graphics quality settings object |
| `unlockedChars` | JSON array of unlocked character emoji |
| `controlBindings` | Custom key remapping per player |

---

## Important Functions Reference

| Function | Line | Purpose |
|----------|------|---------|
| `gameLoop(timestamp)` | 11942 | Main RAF loop; steps physics, updates state, renders |
| `initGame()` | 11225 | Resets all game state and spawns players |
| `startGame()` | 9227 | Reads slots/rules, transitions from menu to game |
| `triggerGameOver(winners)` | 9392 | Ends match, shows results screen |
| `drawGameWorld(map)` | 11862 | Renders platforms, backgrounds, effects |
| `drawWorldEffects()` | 11286 | Renders particles, puddles, screen effects |
| `serializeState()` | 3307 | Serializes full game state for online sync |
| `applyState(state)` | ~3350 | Applies received state on guest side |
| `spawnParticles(x,y,color,count)` | 10760 | Spawns visual particles from pool |
| `canHit(attacker, target)` | 9594 | Team/friendly fire check before damage |
| `registerHit(attacker, target)` | 11119 | Records combo hit, triggers passive hooks |

---

## Workflow Notes for AI Assistants

- **Always read the relevant section** before modifying — the file is large and functions are densely packed
- **Character logic is data-driven**: add/modify characters by editing `CHAR_STATS`, not the core engine
- **Never introduce external dependencies** — the project is intentionally zero-dependency (except CDN PeerJS)
- **Test in browser after every change** — there is no automated test suite
- **Avoid restructuring the file** — the single-file architecture is intentional for distribution
- **The `_`-prefixed fields on Player** are passive ability state; serialize them in `serializeState()` if adding new ones for online play
- **Line 4219 comment** (`// ORIGINAL GAME CODE (unchanged)`) is legacy and can be ignored
- **CSS changes**: the design system uses CSS custom properties; prefer modifying `--ui-*` vars or adding new `@keyframes` rather than inline styles
- **When adding maps**: test all platform coordinates at the 1200×800 canvas scale; the camera zooms dynamically via `camScale`
