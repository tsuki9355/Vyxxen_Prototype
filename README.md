<div align="center">

# VYXXEN — Offshore Run

**A low-poly isometric flight shooter for the browser, built in Three.js.**

Thread an interceptor through an offshore oil-field course: clear turrets and interceptors, skim green platforms for fuel, and make it home to the runway.

<img width="2559" height="1267" alt="image" src="https://github.com/user-attachments/assets/210f66fa-e10b-4e74-9ba4-94923489c46d" />


## Run locally

![Three.js](https://img.shields.io/badge/Three.js-r180-000000?logo=threedotjs&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES_Modules-F7DF1E?logo=javascript&logoColor=black)
![WebGL 2](https://img.shields.io/badge/WebGL-2-990000?logo=webgl&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-6-646CFF?logo=vite&logoColor=white)
![Playwright](https://img.shields.io/badge/Tested_with-Playwright-2EAD33?logo=playwright&logoColor=white)

<!-- Add a screenshot or GIF here, e.g. docs/flight.png -->
<!-- ![Vyxxen in flight](docs/flight.png) -->

</div>

---

## About

Vyxxen Prototype is a browser-based take on the classic isometric shooter *Zaxxon*, and a companion to the Unreal Engine 5 / C++ version of Vyxxen. It was built as an **agentic prototyping experiment**:

1. **Design first.** 40 design decisions, gathered over five rounds of requirements, were written up as a structured Markdown spec ([`design.md`](../design.md)) before any code was written.
2. **Agent-built.** An AI coding agent turned the spec into a playable game, directed and reviewed against the design.
3. **Verified.** The core rules are covered by unit tests, and the full game is checked in a real WebGL browser with Playwright.

Every model, marking, texture and sound is **generated in code**. There are no external art assets, no backend and no accounts.

## Features

- ✈️ **Three-band flight.** Steer left and right, and climb or dive through low, mid and high altitude bands, with an altitude marker and a ground shadow so you can read your height.
- ⛽ **Fuel management.** A full tank lasts about 69 seconds. Hover low over a green pad to refuel automatically, or leave early with a partial tank.
- 🎯 **Combat.** Turrets fire aimed rounds and interceptors make crossing passes. Cargo crates are optional targets, and towers and barriers are solid hazards.
- 💎 **Weapon upgrade.** Violet diamonds add two extra shots at ±22°. You keep the upgrade until you die.
- 🚩 **Checkpoints and lives.** Three sectors, two checkpoints and three lives. Respawns come with a two-second shield.
- 🎮 **Keyboard and gamepad.** Standard gamepads are supported, with a rescaled stick deadzone.
- 🔊 **Synthesized audio.** The engine hum and all effects are generated with the Web Audio API.

## Under the hood

| Area | Highlights |
| --- | --- |
| **Gameplay loop** | `dt` is capped per frame and split into fixed **120 Hz substeps**, so fast bullets and collisions register reliably on slower displays |
| **Collision** | Box checks that include altitude for structures and pads, and sphere-distance checks for bullets, interceptors and pickups |
| **3D math** | Normalized aim vectors for turrets, trigonometry for the spread shot, quaternion-oriented support beams, and a skewed route axis for the isometric course |
| **Rendering** | Physically based `MeshStandardMaterial` (roughness/metalness), emissive effects, ACES filmic tone mapping, sRGB output and distance fog |
| **Shadows** | 2048² PCF soft shadow maps that follow the ship, with depth and normal bias tuned to prevent shadow acne |
| **Performance** | Materials cached by key to avoid duplicate shader programs, geometry shared across bullets and particles, pixel ratio capped at 2, and distance-culled updates |
| **Architecture** | Game rules are kept in a separate testable module (`rules.js`), apart from the rendering (`world.js`) and the game loop (`game.js`) |

## Getting started

**Requirements:** Node.js **20.19+** or **22.12+**, and a desktop browser with WebGL 2.

```sh
cd Vyxxen_Prototype
npm install
npm run dev
```

Open the URL Vite prints (usually **http://127.0.0.1:5173**), then press **Enter** or click **Launch Mission**.

> Opening `index.html` directly won't work. The game has to be served by Vite or another web server.

| Command | What it does |
| --- | --- |
| `npm run dev` | Start the dev server with hot reload |
| `npm run build` | Build production files into `dist/` |
| `npm run preview` | Serve the production build locally |

## Controls

| Action | Keyboard | Gamepad |
| --- | --- | --- |
| Move left / right | `A` / `D` or `←` / `→` | Left stick ↔ |
| Climb / dive | `W` / `S` or `↑` / `↓` | Left stick ↕ |
| Fire (hold) | `Space` | A / Cross |
| Slow flight (hold) | `Shift` | Either trigger |
| Pause / resume | `Esc` or `P` | Start / Menu |
| Launch / replay | `Enter` | Start / Menu |

The sound toggle is in the top-right corner. Audio starts after your first interaction. If your controller isn't detected, press any button on it while the page is focused. The game pauses automatically when the window loses focus.

## How to play

- **Fly the course.** The 6,600-unit route takes about 5½ minutes at cruise speed. Edge beacons mark the flight corridor.
- **Refuel.** Green platforms are about 35–42 seconds apart. Line up with the pad, drop into the low band, and the ship hovers and refills. An empty tank fills in two seconds.
- **Clear threats.** Turrets fire slow aimed rounds, and you can hit their energy masts from any altitude. Interceptors don't shoot, but they will ram you.
- **Survive.** A collision, a hostile round or an empty tank costs a life. You respawn at the last checkpoint with your fuel and a short shield.
- **Score.**

  | Target | Points |
  | --- | --- |
  | Interceptor | 250 |
  | Turret | 150 |
  | Cargo crate | 100 |
  | Mission complete | 1,000 |

<details>
<summary><b>Design interpretations</b></summary>

<br>

- `design.md` asks both for keeping the weapon state on respawn and for losing the upgrade on death. This prototype follows the explicit pickup rule: **death resets you to a single shot**, but your fuel is kept.
- Fuel doesn't drain during the respawn shield. Empty-tank respawns place you over a checkpoint pad, so you can't get stuck dying again and again.
- Destroyed targets and collected pickups stay gone for the rest of the run, even after a respawn. Enemies ahead of the checkpoint are rearmed.
- Restarting resets the whole mission and picks new pickup locations, one per sector. Time spent paused isn't counted in your results.

</details>

## Testing

```sh
npm test                       # Unit tests for the core rules (Node test runner)
npx playwright install chromium
npm run test:browser           # End-to-end WebGL tests in Chromium
```

**Unit tests:** checkpoint progression, life loss and respawn shields, safe dry-tank respawns, fuel drain and refill, exact symmetric 22° spread vectors, curated pickup randomness, course length and pad spacing, and collision with altitude.

**Browser tests:** keyboard flight, firing and pause/restart; refueling; pickups and target damage; checkpoint recovery and fuel depletion; completion and game over; simulated gamepad input; and layout in short desktop windows. Screenshots of the title and flight screens are saved to `test-results/`.

## Project structure

```
Vyxxen_Prototype/
├── index.html          # Page shell, HUD and menu markup
├── src/
│   ├── main.js         # Entry point
│   ├── game.js         # Game loop, input, combat, lifecycle, HUD
│   ├── world.js        # Three.js scene, procedural models, lighting, effects
│   ├── course.js       # Authored hazards, platforms, enemies, pickup candidates
│   ├── rules.js        # Tuning constants and pure, testable game rules
│   ├── audio.js        # Synthesized engine and event sounds
│   └── style.css       # Title screen, flight instruments, results
└── tests/
    ├── rules.test.js   # Unit tests
    └── browser/
        └── game.spec.js  # Playwright integration tests
```

## Credits

Design and direction by **Angus Chou**. Built with [Three.js](https://threejs.org/), [Vite](https://vitejs.dev/) and [Playwright](https://playwright.dev/). Inspired by Sega's *Zaxxon* (1982). UI fonts come from Google Fonts, with local fallbacks.
