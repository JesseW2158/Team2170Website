# 🚗 Bruno Simon's Folio 2025 — Complete Project Guide

This document explains every part of the project, how it's structured, what each component does, and how to begin editing it for your own purposes.

---

## Table of Contents

1. [What This Project Is](#what-this-project-is)
2. [License](#license)
3. [Tech Stack Overview](#tech-stack-overview)
4. [Project Directory Structure](#project-directory-structure)
   - [Critical Distinction: `static/` vs `resources/`](#-critical-distinction-static-vs-resources)
5. [Configuration & Build Files](#configuration--build-files)
6. [Source Code Architecture](#source-code-architecture)
7. [The Game Loop (Step-by-Step Breakdown)](#the-game-loop-step-by-step-breakdown)
8. [Key Systems & Game Modules](#key-systems--game-modules)
9. [Static Assets Overview](#static-assets-overview)
10. [The Compression Pipeline (GLB & Texture Optimization)](#the-compression-pipeline-glb--texture-optimization)
11. [How to Set Up & Run Locally](#how-to-set-up--run-locally)
12. [How to Edit for Your Own Purposes](#how-to-edit-for-your-own-purposes)
13. [Environment Variables](#environment-variables)
14. [Key Dependencies](#key-dependencies)

---

## What This Project Is

This is **Bruno Simon's 2025 creative portfolio** — a fully interactive 3D world rendered in the browser. Users drive a car around an open environment, discovering the creator's projects, career history, easter eggs, secrets, and more. It's essentially **a video game as a portfolio website**.

The project was originally built by [Bruno Simon](https://bruno-simon.com/) (of [Three.js Journey](https://threejs-journey.com/) fame) and open-sourced under the MIT license. The upstream repository is: **https://github.com/brunosimon/folio-2025**

---

## License

```text
MIT License — Copyright (c) 2025 Bruno Simon
```

You are free to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of this software, provided you include the copyright notice. This means **you can build your own portfolio on top of this codebase**, which is likely why you're here.

---

## Tech Stack Overview

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **3D Rendering** | [Three.js](https://threejs.org/) (v0.183+) via `three/webgpu` | Renders the entire 3D world using WebGPU (with WebGL fallback) |
| **Shading Language** | [TSL](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language) | Three.js Shading Language — enables cross-API shader code (WebGL + WebGPU) |
| **Physics** | [Rapier3D](https://rapier.rs/) (`@dimforge/rapier3d`) | Physics engine for vehicle dynamics, collisions, and object interactions |
| **Audio** | [Howler.js](https://howlerjs.com/) | Spatial audio, music playback, sound effects |
| **Animation** | [GSAP](https://gsap.com/) | Smooth camera transitions, UI animations, tweening |
| **Build Tool** | [Vite](https://vitejs.dev/) (v7.x) | Development server, production bundling, HMR |
| **Styling** | [Stylus](https://stylus-lang.com/) | CSS preprocessor for UI styling (`.styl` files) |
| **Camera Controls** | [camera-controls](https://github.com/yomotsu/camera-controls) | Orbit-style camera control for the view |
| **UI Controls** | [Tweakpane](https://tweakpane.github.io/) (v4.x) | Debug/options panels |
| **Model Compression** | [glTF-Transform CLI](https://gltf-transform.dev/cli) + [KTX-Software](https://github.com/KhronosGroup/KTX-Software) | Compresses GLB models (Draco + ETC1S/KTX2) and texture files |
| **Image Processing** | [sharp](https://sharp.pixelplumbing.com/) | Converts UI PNGs to WebP |
| **Deterministic RNG** | [seedrandom](https://github.com/davidbau/seedrandom) | Seeded random numbers for reproducible world state |

---

## Project Directory Structure

```
c:\Github\Team2170Website/
│
├── .gitignore                  # Ignores node_modules, dist, .env, .DS_Store, etc.
├── .npmrc                      # Sets npm --force flag (needed for dependency conflicts)
├── license.md                  # MIT License text
├── package.json                # NPM dependencies and scripts
├── package-lock.json           # Locked dependency versions
├── readme.md                   # Original README from Bruno Simon
├── vite.config.js              # Vite build/dev configuration
│
├── resources/                  # Raw source design assets (NOT served to the browser)
│   ├── folio-2025.blend        # Main Blender scene file
│   ├── folio-2025.blend1       # Blender backup file
│   ├── palette.png             # Color palette reference
│   ├── references-inspiration.pur  # PureRef mood board
│   ├── slabs.sbs               # Substance Designer source
│   ├── stars.psd               # Photoshop star texture source
│   ├── terrain.psd             # Photoshop terrain texture source
│   ├── models/                 # Raw model files
│   ├── renders/                # Pre-rendered images/videos
│   ├── sounds/                 # Raw audio files
│   └── textures/               # Raw texture files
│
├── scripts/                    # Build/utility scripts
│   └── compress.js             # Compresses GLB models & textures (npm run compress)
│
├── sources/                    # SOURCE CODE (Vite root directory)
│   ├── index.html              # Main HTML entry point
│   ├── index.js                # JavaScript entry point
│   ├── threejs-override.js     # Monkey-patches THREE.Object3D.prototype.copy
│   ├── data/                   # Static data definitions
│   │   ├── achievements.js     # Achievement definitions (goals, titles, descriptions)
│   │   ├── consoleLog.js       # Fancy console ASCII art/branding log
│   │   ├── countries.js        # Country list for whisper flags
│   │   ├── lab.js              # Lab area project data
│   │   ├── projects.js         # Portfolio project entries (title, images, URLs, awards)
│   │   └── social.js           # Social media links definition
│   ├── Game/                   # GAME ENGINE — all core systems
│   │   ├── Game.js             # Main Game singleton — initialization orchestrator
│   │   ├── Achievements.js     # Achievement tracking system
│   │   ├── Audio.js            # Audio manager (music, SFX, spatial audio)
│   │   ├── ClosingManager.js   # Handles page/tab close events
│   │   ├── Debug.js            # Debug mode (Tweakpane panels)
│   │   ├── Easter.js           # Easter egg system (date-triggered)
│   │   ├── Events.js           # Custom event emitter/message bus
│   │   ├── Explosions.js       # Explosion physics & visual effects
│   │   ├── Fog.js              # Fog rendering system
│   │   ├── InputFlag.js        # Country flag selection UI component
│   │   ├── InstancedGroup.js   # GPU instanced rendering for many identical objects
│   │   ├── InteractivePoints.js # Points/hints that appear when near interactable objects
│   │   ├── KonamiCode.js       # Konami Code easter egg (↑↑↓↓←→←→BA)
│   │   ├── Lighting.js         # Scene lighting management
│   │   ├── Map.js              # Mini-map UI overlay
│   │   ├── Materials.js        # Custom Three.js materials (MeshDefault, MeshGrid)
│   │   ├── Menu.js             # Main pause/settings menu UI
│   │   ├── Modals.js           # Modal dialog system (map, discord, circuit end)
│   │   ├── Monitoring.js       # Performance/stats monitoring (stats-gl)
│   │   ├── Noises.js           # Procedural noise generation (Perlin/Simplex-like)
│   │   ├── Notifications.js    # Toast/popup notification system
│   │   ├── Objects.js          # Object lifecycle and reset manager
│   │   ├── Options.js          # User options (audio, quality, respawn, reset)
│   │   ├── Overlay.js          # Screen overlay effects
│   │   ├── Player.js           # Player state (position, velocity, vehicle relationship)
│   │   ├── PreRenderer.js      # Pre-rendering warmup to avoid first-frame stutter
│   │   ├── Quality.js          # Quality level detection and management
│   │   ├── RayCursor.js        # 3D raycasting for mouse/touch interaction
│   │   ├── References.js       # Reference transform lookups (for aligning objects)
│   │   ├── Rendering.js        # WebGPU/WebGL renderer setup and post-processing
│   │   ├── ResourcesLoader.js  # Async resource loader with progress tracking
│   │   ├── Respawns.js         # Respawn point system
│   │   ├── Reveal.js           # Gradual world reveal mechanic (fades in as you explore)
│   │   ├── Server.js           # Optional server connection (whispers, circuit leaderboard)
│   │   ├── Tabs.js             # UI tab component
│   │   ├── Terrain.js          # Terrain mesh generation and management
│   │   ├── TextCanvas.js       # Canvas-based text rendering for 3D textures
│   │   ├── Ticker.js           # Game tick/scheduler system (runs steps at intervals)
│   │   ├── Time.js             # Game time tracking (delta, elapsed, frame)
│   │   ├── Title.js            # Title/heading display
│   │   ├── Tornado.js          # Tornado physics and path system
│   │   ├── Tracks.js           # Tire track decals on terrain
│   │   ├── Trails.js           # Vehicle trail effects
│   │   ├── View.js             # Camera/view management
│   │   ├── Viewport.js         # Viewport/screen resize management
│   │   ├── Water.js            # Water surface rendering
│   │   ├── Weather.js          # Weather system (rain, snow, lightning)
│   │   ├── Wind.js             # Wind system (direction, strength, particle effects)
│   │   └── Zones.js            # Zone/trigger detection system
│   │
│   ├── Game/BlackFriday/       # Black Friday event system
│   │   ├── BlackFriday.js
│   │   └── FragmentObject.js
│   │
│   ├── Game/Cycles/            # Time-of-day and seasonal cycle systems
│   │   ├── Cycles.js           # Base cycle manager
│   │   ├── DayCycles.js        # Day/night cycle (sun position, sky colors)
│   │   └── YearCycles.js       # Seasonal cycle (spring, summer, fall, winter)
│   │
│   ├── Game/Geometries/        # Custom Three.js BufferGeometry classes
│   │   ├── LineGeometry.js
│   │   ├── PortalSlabsGeometry.js
│   │   └── WindLineGeometry.js
│   │
│   ├── Game/Inputs/            # ALL input handling systems
│   │   ├── Inputs.js           # Central input manager (aggregates all input sources)
│   │   ├── Gamepad.js          # Xbox/PlayStation gamepad support
│   │   ├── InteractiveButtons.js  # Dynamic context-sensitive action buttons
│   │   ├── Keyboard.js         # Keyboard input (WASD, arrows, shift, space, etc.)
│   │   ├── Nipple.js           # Virtual joystick for mobile touch
│   │   ├── Pointer.js          # Mouse/touch pointer for camera drag
│   │   └── Wheel.js            # Mouse wheel zoom
│   │
│   ├── Game/Materials/         # Custom Three.js shader materials
│   │   ├── MeshDefaultMaterial.js  # Default PBR-ish material with palette support
│   │   └── MeshGridMaterial.js     # Grid/wireframe rendering material
│   │
│   ├── Game/Passes/            # Post-processing render passes
│   │   └── cheapDOF.js         # Cheap depth-of-field effect
│   │
│   ├── Game/Physics/           # Rapier3D physics wrapper
│   │   ├── Physics.js          # Main physics world setup
│   │   ├── PhysicsVehicle.js   # Vehicle physics (suspension, wheels, engine)
│   │   └── PhysicsWireframe.js # Debug wireframe overlay for physics colliders
│   │
│   ├── Game/utilities/         # Shared utility modules
│   │   ├── maths.js            # Math helpers
│   │   ├── ObservableMap.js    # Reactive Map data structure
│   │   ├── ObservableSet.js    # Reactive Set data structure
│   │   └── time.js             # Time formatting utilities
│   │
│   └── Game/World/             # WORLD — all visual & interactive world objects
│       ├── World.js            # World manager — step scheduler for rendering order
│       ├── Intro.js            # Intro/loading screen
│       ├── Floor.js            # Terrain floor slabs
│       ├── Grass.js            # Grass rendering
│       ├── Benches.js          # Interactive benches (sit/knock over)
│       ├── Bricks.js           # Brick objects
│       ├── Bubble.js           # Bubble particle effects
│       ├── Bushes.js           # Bush foliage
│       ├── Confetti.js         # Confetti particle effect
│       ├── ExplosiveCrates.js  # Destructible explosive crate objects
│       ├── Fences.js           # Fence objects
│       ├── Fireballs.js        # Fireball effects
│       ├── Flowers.js          # Flower objects
│       ├── Foliage.js          # General foliage rendering
│       ├── Grid.js             # Grid/debug overlay
│       ├── Lanterns.js         # Lantern light objects
│       ├── Leaves.js           # Falling leaves particle system
│       ├── Lightnings.js       # Lightning strike effects (during storms)
│       ├── PoleLights.js       # Pole light objects
│       ├── RainLines.js        # Rain particle system
│       ├── Scenery.js          # Background scenery/mountains
│       ├── Snow.js             # Snow particle system
│       ├── Trees.js            # Tree management (birch, oak, cherry)
│       ├── VisualTornado.js    # Tornado visual rendering
│       ├── VisualVehicle.js    # Vehicle visual rendering (separate from physics body)
│       ├── WaterSurface.js     # Water surface with reflections
│       ├── Whispers.js         # Visitor message bubbles in 3D space
│       ├── WindLines.js        # Wind visualization lines
│       └── Areas/              # INTERACTIVE ZONES — each a unique location/experience
│           ├── Areas.js        # Zone manager (load/unload based on proximity)
│           ├── Area.js         # Base area/zone class
│           ├── AchievementsArea.js  # Achievement display area
│           ├── AltarArea.js         # Altar/shrine area
│           ├── BehindTheSceneArea.js # Tech stack/devlog info area
│           ├── BowlingArea.js       # Bowling mini-game area
│           ├── CareerArea.js        # Career timeline area
│           ├── CircuitArea.js       # Racing circuit area (time trials)
│           ├── CookieArea.js        # Cookie clicker area
│           ├── EasterArea.js        # Easter egg location
│           ├── LabArea.js           # Lab/experiments area
│           ├── LandingArea.js       # Starting/spawn area
│           ├── ProjectsArea.js      # Portfolio projects showcase
│           ├── SocialArea.js        # Social media links area
│           ├── TimeMachineArea.js   # Time machine (shows old portfolio versions)
│           └── ToiletArea.js        # Toilet humor area
│
│     └── style/                # Stylus CSS stylesheets (UI styling)
│         ├── index.styl        # Main style entry (imports all others)
│         ├── general.styl      # General/base styles
│         ├── fonts.styl        # Font loading and definitions
│         ├── menu.styl         # Menu/pause screen styles
│         ├── modals.styl       # Modal dialog styles
│         ├── map.styl          # Mini-map styles
│         ├── notifications.styl # Toast notification styles
│         ├── options.styl      # Options panel styles
│         ├── controls.styl     # Controls display panel styles
│         ├── achievements.styl # Achievement panel styles
│         ├── circuit.styl      # Circuit/racing UI styles
│         ├── circuit-end.styl  # Race finish modal styles
│         ├── whispers.styl     # Whisper input form styles
│         ├── tooltips.styl     # Tooltip styles
│         ├── tabs.styl         # Tab component styles
│         ├── tweakpane.styl    # Debug Tweakpane panel styling
│         ├── interactiveButtons.styl # Context action button styles
│         ├── behindTheScene.styl    # Behind the scene panel styles
│         ├── discord.styl      # Discord modal styles
│         ├── blackFriday.styl  # Black Friday event styles
│         └── easter.styl       # Easter event styles
│
└── static/                     # PUBLIC STATIC ASSETS (served directly by Vite)
    ├── palette.png / palette.ktx  # Color palette texture
    ├── achievements/           # Achievement UI textures/glyphs
    ├── areas/                  # Area-specific textures (e.g., satanStar)
    ├── basis/                  # Basis Universal compressed textures
    ├── behindTheScene/         # Behind the scene assets (stars texture)
    ├── benches/                # Bench GLB models
    ├── birchTrees/             # Birch tree GLB models (visual + references)
    ├── blackFriday/            # Black Friday event assets
    ├── bricks/                 # Brick GLB models
    ├── bushes/                 # Bush GLB models (references for instancing)
    ├── career/                 # Career timeline textures
    ├── cherryTrees/            # Cherry tree GLB models (visual + references)
    ├── christmas/              # Christmas event assets
    ├── draco/                  # Draco-compressed GLB models
    ├── easter/                 # Easter event assets
    ├── explosiveCrates/        # Explosive crate GLB models
    ├── favicons/               # Favicons, app icons, webmanifest
    ├── fences/                 # Fence GLB models
    ├── floor/                  # Floor slab textures
    ├── flowers/                # Flower GLB models (references for instancing)
    ├── foliage/                # Foliage SDF textures
    ├── fonts/                  # Custom font files (e.g., Pally-Medium)
    ├── interactivePoints/      # Key icon textures (Enter, Cross, A button)
    ├── intro/                  # Intro screen textures
    ├── jukebox/                # Jukebox music note textures
    ├── lab/                    # Lab area textures
    ├── lanterns/               # Lantern GLB models
    ├── oakTrees/               # Oak tree GLB models (visual + references)
    ├── overlay/                # Screen overlay pattern textures
    ├── playground/             # Playground GLB models (visual + physical)
    ├── poleLights/             # Pole light GLB models
    ├── projects/               # Project showcase textures/images
    ├── respawns/               # Respawn reference GLB models
    ├── scenery/                # Scenery/mountain GLB models
    ├── social/                 # Social sharing images
    ├── sounds/                 # ALL audio files (music, SFX)
    ├── terrain/                # Terrain texture and GLB model
    ├── timeMachine/            # Time machine screen textures
    ├── tornado/                # Tornado path reference GLB models
    ├── ui/                     # UI SVGs, WebP images, flag sprites
    ├── vehicle/                # Vehicle/car GLB models
    └── whispers/               # Whisper flame textures
```

---

### 🔑 Critical Distinction: `static/` vs `resources/`

This is one of the most important concepts to understand before editing the project:

| Directory | Purpose | Served to Browser? | Contains |
|-----------|---------|-------------------|----------|
| **`static/`** | PUBLIC asset store | ✅ Yes — copied into `dist/` on build | Final, production-ready GLB models, KTX2 textures, audio files, fonts, UI images |
| **`resources/`** | PRIVATE source materials | ❌ No — never reaches the browser | Master Blender file, Substance Designer graphs, PSD source files, raw/high-res textures, reference models, raw audio, PureRef mood board |

**Rule of thumb:**
- **`resources/`** = your "source of truth" editable files (`.blend`, `.psd`, `.sbs`, raw uncompressed textures, pre-production audio)
- **`static/`** = the exported, compressed, browser-ready versions of those same assets (`.glb`, `.ktx`, `.webp`, `.wav`/`.mp3`)

### How assets flow from `resources/` to `static/`

The typical workflow for adding or modifying any visual asset:

```
resources/folio-2025.blend        (edit in Blender)
         │
         ▼  Export GLB (uncompressed, with palette texture node muted)
static/vehicle/vehicle.glb        (raw export)
         │
         ▼  npm run compress      (scripts/compress.js)
static/vehicle/vehicle-compressed.glb   (Draco geometry + ETC1S/KTX2 textures)
```

```
resources/textures/someTexture.png   (high-res source from Substance/Photoshop)
         │
         ▼  Export/resize for game use
static/someFolder/someTexture.png    (game-resolution version)
         │
         ▼  npm run compress          (toktx + sharp)
static/someFolder/someTexture.ktx    (GPU-compressed KTX2)
```

### Where to put new models

| If you want to... | Put it here | Why |
|-------------------|-------------|-----|
| **Add a new 3D object to the world** | Export GLB to `static/<category>/` | The game loads models from `static/` at runtime via `ResourcesLoader.js` |
| **Keep the editable master model** | Save `.blend` or source files in `resources/models/` | So you (or others) can re-edit it later |
| **Add a new texture/material** | Export PNG to `static/<category>/` | Referenced by GLBs or loaded directly by game code |
| **Keep the Substance/Photoshop source** | Save `.sbs`/`.psd` to `resources/textures/` | Editable source, never shipped to browser |
| **Add sound effects or music** | Put final audio in `static/sounds/` | Loaded by `Audio.js` at runtime |
| **Keep raw recording stems** | Put in `resources/sounds/` | Pre-mix/edit source material |

### Why `resources/` is gitignored (mostly) and `static/` is not

- `resources/folio-2025.blend` is tracked in git because it's the master scene. Other `resources/` subfolders (`models/`, `textures/`, `renders/`, `sounds/`) are typically **gitignored** because they contain large binary source files that aren't needed for the build.
- `static/` is fully tracked because without it, the project cannot build or run.

### Concrete Example: The Slabs Texture

The file `resources/slabs.sbs` is a **Substance Designer graph** from the original Bruno Simon project — a 7KB XML document defining a procedural material node network. This is purely a **source file** — the developer opens it in Substance Designer, tweaks parameters, and exports the result as a flat PNG texture that goes into `static/floor/slabs.png`. The `.sbs` file itself never reaches the browser; only the baked PNG (and its KTX2 compressed version) does.

---

## Configuration & Build Files

### `package.json`

- **`name`**: `bruno-simon-folio-2025`
- **`type`**: `module` (ES modules throughout — no CommonJS)
- **Scripts**:
  - `npm run dev` — Starts Vite dev server (hot-reload, `localhost:1234`)
  - `npm run build` — Production build into `dist/` folder
  - `npm run preview` — Preview the production build locally
  - `npm run compress` — Runs the asset compression pipeline (GLB + textures)

### `vite.config.js`

| Setting | Value | Purpose |
|---------|-------|---------|
| `root` | `sources/` | Source files directory (where `index.html` lives) |
| `envDir` | `../` | Environment (`.env`) file location |
| `publicDir` | `../static/` | Static assets served as-is |
| `base` | `./` | Relative public path (works for any domain) |
| `server.host` | `true` | Exposes to local network, shows URL in terminal |
| `server.open` | `true` | Auto-opens browser on start |
| `build.outDir` | `../dist` | Production output directory |
| `build.emptyOutDir` | `true` | Clears dist/ before each build |
| `build.sourcemap` | `false` | No sourcemaps in production |
| **Plugins** | `wasm`, `topLevelAwait`, `restart`, `nodePolyfills` | WASM support for Rapier, restart on static file changes, Node polyfills for browser |

### `.npmrc`

Contains `force = true` — npm will override peer dependency conflicts. This is needed because some dependencies (like `three@0.183.2`) are cutting-edge and have peer dep mismatches.

### `.gitignore`

Ignores: `.DS_Store`, `node_modules/`, `dist/`, `.vercel/`, `.VSCodeCounter/`, `.env*` (all env files), `.autosave/`, `.wav` (but NOT `static/sounds/music/*.wav` which are the CC0 music tracks).

---

## Source Code Architecture

### Entry Points

| File | Role |
|------|------|
| `sources/index.html` | The HTML shell. Contains the canvas element, all UI markup (menu, modals, buttons, navigation panels), font loaders, Google Analytics, meta tags, and CSS/JS imports. References `./index.js` as the module entry. |
| `sources/index.js` | JavaScript entry point. Imports the `threejs-override.js` (monkey-patches), imports the `Game` class, optionally logs console ASCII art, and instantiates `new Game()`. |
| `sources/threejs-override.js` | Overrides `THREE.Object3D.prototype.copy` to properly deep-copy rotation order, quaternion, scale, matrix auto-update settings, render order, shadow settings, animations array, etc. This fixes a Three.js limitation. |

### The Game Singleton (`sources/Game/Game.js`)

The `Game` class is a **singleton** — there is only one instance ever created. Its constructor:

1. Checks if an instance already exists; if so, returns it immediately
2. Calls `this.init()` — an **async** method

### Initialization Flow (`Game.init()`)

The init method orchestrates everything in a carefully ordered sequence:

**Phase 1 — Core Setup (synchronous):**
- Grabs `.game` div and `.js-canvas` element from the DOM
- Creates `Scene`, `Debug`, `ResourcesLoader`, `Quality`, `Server`, `Ticker`, `Time`
- Creates `DayCycles`, `YearCycles`, `Inputs`, `Audio`, `Notifications`, `RayCursor`, `Viewport`, `Modals`, `Menu`
- Creates `Rendering` and awaits renderer setup (WebGPU/WebGL detection)

**Phase 2 — First Resource Batch (async):**
- Loads essential resources needed immediately: respawn references, star texture, intro sound texture, palette texture

**Phase 3 — More Systems (synchronous):**
- `Options`, `Respawns`, `View`, post-processing setup, rendering start
- `Reveal`, `Noises`, `Weather`, `Wind`, `Tracks`, `Lighting`, `Fog`, `Water`, `Materials`, `Objects`, `Explosions`, `World`

**Phase 4 — Second Resource Batch + Rapier (async, parallel):**
- Loads all remaining model/texture resources (vehicle, trees, terrain, etc.) while simultaneously loading the Rapier3D WASM physics engine
- Updates intro progress bar during loading

**Phase 5 — Physics & Final Systems (synchronous):**
- `Terrain`, `Physics`, `PhysicsWireframe`, `PhysicsVehicle`, `Zones`, `Player`, `ClosingManager`, `InteractivePoints`, `KonamiCode`, `Achievements`, `Tornado`, `Map`, `Title`
- Runs `world.step(1)` to kickstart the game loop
- `Overlay`, optional `PreRenderer` warmup

### Game.reset()

Resets the entire world state: respawns player, resets all objects, explosive crates, bowling pins, cookies, toilet cabin, social statue/fans, benches, fences, bricks, lanterns. Uses GSAP for the delayed achievement trigger.

---

## The Game Loop (Step-by-Step Breakdown)

The game follows a **fixed-step tick system** (defined in the readme) where each frame processes systems in a specific numbered order. This ordering ensures data dependencies are satisfied (e.g., physics runs before rendering).

```
Step 0:    Time, Inputs
Step 1:    Player:pre-physics (processes inputs → desired movement)
Step 2:    PhysicalVehicle:pre-physics (processes player intent → vehicle forces)
Step 3:    Physics (Rapier3D step — resolves all physics, collisions)
Step 4:    PhysicsWireframe (debug collider visualization), Objects (update after physics)
Step 5:    PhysicalVehicle:post-physics (reads physics results → vehicle state)
Step 6:    Player:post-physics (reads physics results → player state)
Step 7:    View (camera follows player based on inputs + player state)
Step 8:    Intro, DayCycles, YearCycles, Weather, Zones, VisualVehicle
Step 9:    Wind, Lighting, Tornado, InteractivePoints, Tracks
Step 10:   Areas++, Foliage, Fog, Reveal, Terrain, Trails, Floor, Grass, Leaves,
           Lightnings, RainLines, Snow, VisualTornado, WaterSurface, Benches,
           Bricks, ExplosiveCrates, Fences, Lanterns, Whispers
Step 13:   InstancedGroup (GPU instancing batches — updates all instanced objects at once)
Step 14:   Audio, Notifications, Title
Step 998:  Rendering (WebGPU/WebGL draw call)
Step 999:  Monitoring (performance stats)
```

---

## Key Systems & Game Modules

### Input System (`Game/Inputs/`)
Multi-source input aggregation supporting **keyboard** (WASD, arrows, shift for boost, space for jump, enter to interact), **gamepad** (Xbox and PlayStation layouts), **touch/mobile** (virtual nipple joystick + two-finger camera control), and **mouse pointer** (drag to orbit camera). All funneled through `Inputs.js` and translated into player actions.

### Physics (`Game/Physics/`)
Wraps **Rapier3D** — a Rust-based WASM physics engine. Handles:
- **Vehicle physics** (`PhysicsVehicle.js`): Suspension, wheel friction, engine forces, steering
- **Collision detection**: Objects, terrain, crates, fences, etc.
- **Wireframe debugging** (`PhysicsWireframe.js`): Visualizes invisible physics colliders

### World (`Game/World/`)
Contains all visual objects in the 3D scene:
- **Trees** (`Trees.js`): Three tree types (birch, oak, cherry) with instanced rendering
- **Weather effects**: `RainLines.js`, `Snow.js`, `Lightnings.js`, `Leaves.js`, `WindLines.js`
- **Dynamic objects**: `Benches.js`, `Bricks.js`, `Fences.js`, `Lanterns.js`, `ExplosiveCrates.js`
- **Vehicle visuals** (`VisualVehicle.js`): Separate from physics — handles wheel spin, steering visuals, hydraulics
- **Water** (`Water.js`): Reflective water surface
- **Whispers** (`Whispers.js`): 3D message bubbles left by visitors
- **Tracks/Trails** (`Tracks.js`, `Trails.js`): Tire marks and trails on terrain
- **Intro** (`Intro.js`): Loading/intro screen with progress animation

### Areas (`Game/World/Areas/`)
**Interactive zones** scattered across the world. Each is a unique location with specific interactions:
- **LandingArea**: Starting point
- **ProjectsArea**: Portfolio project showcase (interactive billboards)
- **CareerArea**: Career timeline with images
- **LabArea**: Experiments/work showcase
- **SocialArea**: Social media links
- **CircuitArea**: Racing time-trial track with leaderboard
- **BowlingArea**: Bowling mini-game
- **CookieArea**: Cookie clicker
- **AchievementsArea**: Achievement display
- **BehindTheSceneArea**: Tech info/credits
- **TimeMachineArea**: Shows previous portfolio versions
- **AltarArea**, **EasterArea**, **ToiletArea**: Easter eggs/humor

### Cycles (`Game/Cycles/`)
- **DayCycles**: Sun position, lighting changes, time of day (day → sunset → night → sunrise)
- **YearCycles**: Seasonal changes — spring (cherry blossoms), summer (green), fall (orange/red), winter (snow)

### Rendering (`Game/Rendering.js`)
- Detects WebGPU support; falls back to WebGL
- Sets up post-processing (depth-of-field via `cheapDOF.js`)
- Manages renderer lifecycle

### Data Files (`sources/data/`)
Pure JavaScript data arrays (no JSON — they're JS modules):
- `achievements.js`: Achievement definitions with goals, titles, descriptions, target counts
- `projects.js`: Portfolio project entries (title, URL, role, collaborators, awards, images)
- `lab.js`: Lab project data
- `social.js`: Social media link definitions (X, Bluesky, YouTube, GitHub, Discord, etc.)
- `countries.js`: Country list for whisper flag selection
- `consoleLog.js`: ASCII art console log branding

---

## Static Assets Overview

The `static/` directory is served directly (defined as `publicDir` in Vite config). It contains:

- **GLB 3D models**: All `.glb` files have companion `-compressed.glb` versions (Draco + KTX2 compressed)
- **KTX2 textures**: `.ktx` GPU-friendly compressed textures alongside original `.png` files
- **UI assets**: `.svg` icons, `.webp` images (compressed from PNG)
- **Fonts**: Custom web fonts (Pally-Medium)
- **Audio**: Music tracks and sound effects in various formats
- **Favicons**: Multi-size favicon package

---

## The Compression Pipeline

Run via `npm run compress` (executes `scripts/compress.js`):

### GLB Models
1. Finds all `.glb` files in `static/` (ignoring already-compressed ones)
2. Runs `gltf-transform etc1s` — compresses embedded textures to ETC1S/KTX2 (lossy, GPU-decodable)
3. Runs `gltf-transform draco` — compresses mesh geometry with Draco (edgebreaker method, quantized attributes)
4. Outputs `-compressed.glb` files alongside originals

### Textures
1. Finds all `.png`/`.jpg` files in `static/` (excluding `ui/`, `favicons/`, `social/`)
2. Uses `toktx` (from KTX-Software) with preset flags depending on file path:
   - Most textures: `etc1s --qlevel 255` (ETC1S compression, highest quality)
   - Palette, terrain, career textures: `uastc` (higher quality universal format)
   - Single-channel textures (SDF, glyphs, etc.): `--target_type R --swizzle r001`
3. Outputs `.ktx` or `.ktx2` files

### UI Images
1. Finds all `.png`/`.jpg` in `static/ui/`
2. Uses `sharp` to convert to `.webp` at quality 80

### Required CLI Tools
- `gltf-transform` CLI (npm package `@gltf-transform/cli`)
- `toktx` from [KTX-Software](https://github.com/KhronosGroup/KTX-Software)

---

## `resources/folio-2025.blend` — FRC Field CAD Assembly (2026 Game "Reefscape")

Despite the filename suggesting Bruno Simon's original portfolio scene, this blend file has been **completely replaced** with Team 2170's CAD assembly of the 2026 FIRST Robotics Competition playing field. The original Bruno Simon artistic scene is no longer present in this file.

**Verified Blender metadata (extracted via Blender 5.1.2 CLI):**

| Property | Value |
|---|---|
| Blender version | 5.1.2 (hash `ec6e62d40fa9`, built 2026-05-19) |
| Compression | Zstd (Blender 4.x/5.x native — file begins with `(\xb5/\xfd` magic bytes) |
| Objects | **6,378** total |
| Meshes | **313** unique mesh data blocks |
| Materials | **30** (flat RGB colors named by channel values — typical of CAD import) |
| Collections | **1** — `FE-2026: REBUILT™ Playing Field` |
| Cameras | 0 |
| Lights | 0 |
| Armatures | 0 |
| Actions | 0 |
| Images | 2 (embedded in file) |

**Object breakdown:**
- **5,083 objects** prefixed with `FE-` (Field Elements): Rail assemblies, weldments, driver station supports, corner braces, ramps, aluminum pipes, plates, tubing, angles, rivets, hook-and-loop tape strips, connector pins — all individually modeled with engineering precision from FRC field CAD.
- **1,295 objects** with generic names: `3/16in Rivet, 0.251-0.375in Grip Length`, `Hook Tape, 3in Wide, 72in Long`, `Loop Tape, 2in Wide, 3in Long`, `Configurable Guardrail`, fuel elements (`GE-26900: Fuel`) — all FRC field components.
- Every object uses an EMPTY→MESH instance chain (CAD assembly hierarchy pattern: each physical part is an Empty parent with a Mesh child, with interleaved `occurrence of ...` Empty instances for sub-assemblies).

**Top-level assemblies observed:**
- `FE-2026-02: Welded Perimeter Assy` — The outer field wall/perimeter
- `FE-00014: Field Entry Ramp` — Entry ramps with hinges, diamond plate, channel, angle brackets
- `FE-00022: Field Rail Assy, Middle` — Mid-field rail assemblies with pipe weldments
- `FE-00048: Driver Station Support Assy` — Driver station support weldments
- `FE-00061: Corner Support, Left Assembly` — Corner bracing with tubes, rods, bars, tape strips
- `GE-26900: Fuel` — Fuel elements (450+ instances)
- `Configurable Guardrail` — Adjustable guardrail assemblies

**Why this file exists in the project:**
The file was retained because the project originated as a fork of `brunosimon/folio-2025`, and the `resources/` directory was repurposed to hold Team 2170's CAD source file. It is **not used at runtime** by the website — it is a reference/source file for the FRC field model. The actual browser-served assets are the glTF models in `static/`.

**To re-export models from this file:** Open it in Blender 5.x, select the desired assembly/mesh, export as GLB to the appropriate `static/` subdirectory, then run `npm run compress` to optimize for web delivery.

---

### `resources/references-inspiration.pur` — Reference Board
A PureRef (`.pur`) file — a visual reference/mood board tool. This was part of Bruno Simon's original `folio-2025` project. Preserved from the upstream fork, not used by this website.

### `resources/palette.png` — Color Palette Reference
A color palette image used as a reference by the original folio-2025 project. A KTX-compressed copy also exists at `static/palette.ktx`.

### `resources/slabs.sbs` — Substance Designer File
An Allegorithmic Substance Designer (`.sbs`) source file — a procedural material authoring graph. Part of Bruno Simon's original material pipeline. The 7KB XML document defines a node network (flood fill, slope blur, tile generator, cell noise, HBAO, edge detect, level adjustments, blend operations) that generates the floor slab texture.

### `resources/stars.psd` & `resources/terrain.psd` — Photoshop Source Files
Adobe Photoshop (`.psd`) source files for the starfield and terrain textures from the original folio-2025 project. These are layered source files from which the final compressed textures in `static/` were exported.

### `resources/models/`, `resources/renders/`, `resources/sounds/`, `resources/textures/`
Subdirectories from the original Bruno Simon project structure. May contain intermediate models, render previews, raw audio stems, and texture source files that fed into the `static/` output assets. These are not consumed directly by the website at runtime.

**Note on all `resources/` files:** Only `folio-2025.blend` was actively repurposed by Team 2170 (replacing the original scene with the FRC field CAD assembly). The remaining files are Bruno Simon's original artistic sources, preserved from the fork but unused by the Team 2170 website.

---

## How to Set Up & Run Locally

### Prerequisites
- **Node.js** (latest LTS recommended) — [Download](https://nodejs.org/en/download/)
- **Git** (to clone)
- **PowerShell/Bash terminal**

### Step-by-Step Setup

```bash
# 1. Clone the repository
git clone https://github.com/JesseW2158/Team2170Website.git
cd Team2170Website

# 2. Create .env file (see Environment Variables section below)
#    At minimum, create an empty .env or copy from example

# 3. Install dependencies (--force is needed due to peer dep conflicts)
npm install --force

# 4. Start development server
npm run dev
# → Opens browser at localhost:1234 with hot-reload
```

### Additional Commands

```bash
# Production build (outputs to dist/)
npm run build

# Preview production build locally
npm run preview

# Compress static assets (only needed if you add new GLB/textures)
npm run compress
```

---

## How to Edit for Your Own Purposes

This is a step-by-step guide to turning this project into **your own** portfolio.

### 1. Fork/Clone & Clean Up

- Fork the repository or clone it into a new repo
- Update `package.json` — change `name`, `author`, `description`
- Replace `readme.md` with your own README

### 2. Change Personal Information

All personal data lives in `sources/data/`:

| File | What to Change |
|------|---------------|
| `projects.js` | Replace Bruno's projects with **your own** projects. Each entry has `title`, `url`, `attributes` (role, collaborators), `distinctions` (awards), and `images`. |
| `lab.js` | Replace lab experiments with your own work |
| `social.js` | Update all social media links to **your** profiles (X, Bluesky, YouTube, GitHub, LinkedIn, Discord, email) |
| `achievements.js` | Modify achievement goals to match your world layout |
| `countries.js` | Customize or trim the country list |
| `consoleLog.js` | Replace ASCII art with your own branding |

### 3. Rebrand the UI

In `sources/index.html`:

- **Title tag**: `<title>Your Name</title>`
- **Meta tags**: Update all `og:`, `twitter:`, `itemprop` meta tags with your name, description, URL, and share image
- **Google Analytics**: Replace the GA tag (`G-JMSN30BQ5J`) with your own, or remove the analytics script block entirely
- **Home content** (line 225-232): Replace Bruno's intro text with your own bio
- **Behind the scene content** (line 682-730): Update tech stack notes, devlog links, credits with your own information
- **Share image**: Replace `static/social/share-image.png` with your own

### 4. Customize the 3D World

**The Blender file** (`resources/folio-2025.blend`) currently contains an FRC field CAD assembly (see the detailed analysis in the section above). For a portfolio project, you'll likely want to replace this with your own 3D scene:

1. Open the blend file in Blender 5.x+
2. Either modify the existing field scene, or — more practically for a portfolio — start fresh by importing your own models, terrain, and layouts
3. Export individual GLB files to the appropriate `static/` subdirectories
4. Run `npm run compress` to compress the new models

**Alternatively** — you can modify the source code without using Blender:

- **Areas**: Each area in `sources/Game/World/Areas/` can be modified. Remove areas you don't want by deleting their files and removing their imports in `Areas.js`.
- **Vehicle**: Replace `static/vehicle/*.glb` with a different vehicle model
- **Trees/Foliage**: Replace tree models or modify `Trees.js`
- **Terrain**: Replace the terrain GLB and texture

### 5. Change the Environment Variables

Create/modify `.env`:

| Variable | Purpose |
|----------|---------|
| `VITE_PLAYER_SPAWN` | Starting spawn point name (default: `landing`) |
| `VITE_COMPRESSED` | Use compressed assets (`true`/`false`) |
| `VITE_GAME_PUBLIC` | Expose `window.game` globally for debugging |
| `VITE_LOG` | Show console ASCII art on load |
| `VITE_ANALYTICS_TAG` | Google Analytics measurement ID |

### 6. Styling Changes

All UI styling is in `sources/style/*.styl` (Stylus preprocessor). Modify:
- `general.styl` — Colors, fonts, base styles
- `menu.styl` — Navigation menu appearance
- `achievements.styl`, `circuit.styl`, etc. — Individual panel styles
- `fonts.styl` — Font definitions (the project uses Nunito, Amatic SC, and Pally-Medium)

### 7. Music & Audio

- Replace music files in `static/sounds/musics/` with your own audio
- Sound effects in `static/sounds/` can be swapped
- Modify `Audio.js` to change playback logic, volumes, spatial audio settings

### 8. Removing Unwanted Features

Features you might want to remove:

| Feature | Files to Remove/Modify |
|---------|----------------------|
| Gamepad support | Remove `Inputs/Gamepad.js`, remove import in `Inputs.js` |
| Mobile touch joystick | Remove `Inputs/Nipple.js` |
| Racing circuit/leaderboard | Remove `World/Areas/CircuitArea.js`, circuit modal in HTML |
| Whispers/visitor messages | Remove `World/Whispers.js`, whisper UI in HTML |
| Server connectivity | Remove `Server.js` |
| Achievements system | Remove `Achievements.js`, data/achievements.js |
| Easter eggs (Konami Code, seasonal events) | Remove `KonamiCode.js`, `Easter.js`, `BlackFriday/` |
| Tornado | Remove `Tornado.js`, `World/VisualTornado.js`, `tornado/` static assets |
| Bowling mini-game | Remove `World/Areas/BowlingArea.js` |
| Debug/Tweakpane | Remove `Debug.js` |

### 9. HTML Structure Customization

`index.html` contains all UI markup. Key sections:

- **`.js-menu`** (lines 102-738): Main menu with navigation, previews, and content panels. Modify text, add/remove navigation items, change content.
- **`.js-modals`** (lines 742-878): Modal dialogs (circuit end, Discord, map). Remove or customize.
- **`.js-notifications`** (lines 882-884): Toast notification container.
- **`.js-touch-buttons`** (lines 67-77): Mobile touch control buttons.

### 10. Deployment

```bash
npm run build
# → outputs to dist/ folder
```

Deploy the `dist/` folder to any static host:
- **Vercel**: Just connect the repo (was used originally)
- **Netlify**: Connect repo, set build command to `npm run build`, publish directory to `dist`
- **GitHub Pages**: Use a deploy action
- **Any static server**: Serve `dist/` directory

**Important**: The server component (`Server.js`) is for whispers and circuit leaderboards. The portfolio works fully offline without it. Bruno did not open-source the server code.

---

## Environment Variables

Create a `.env` file in the project root (same level as `package.json`):

```bash
# .env
VITE_PLAYER_SPAWN=landing       # Which respawn point to start at (see Respawns.js)
VITE_COMPRESSED=true            # Use draco+ktx compressed assets (true = compressed, false = uncompressed)
VITE_GAME_PUBLIC=true           # Expose window.game for browser console debugging
VITE_LOG=true                   # Show console ASCII art branding
VITE_ANALYTICS_TAG=G-XXXXXXXXXX # Google Analytics tag (remove scripts in HTML if not needed)
```

The `VITE_COMPRESSED` flag controls asset loading throughout `Game.js`:
- `true` → uses `-compressed.glb` files and `.ktx` textures
- `false`/`undefined` → uses uncompressed `.glb` and `.png` textures

For development where you're editing assets frequently, set to `false` to avoid needing to recompress after every change.

---

## Key Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `three` | ^0.183.2 | 3D rendering (WebGPU-first via `three/webgpu`) |
| `@dimforge/rapier3d` | ^0.17.3 | Physics engine (vehicle, collisions) |
| `gsap` | ^3.12.5 | Animation/tweening |
| `howler` | ^2.2.4 | Audio playback |
| `vite` | ^7.2.4 | Build tool & dev server |
| `stylus` | ^0.64.0 | CSS preprocessor for `.styl` files |
| `camera-controls` | ^3.1.2 | Camera orbit controls |
| `tweakpane` | ^4.0.4 | Debug settings panel |
| `seedrandom` | ^3.0.5 | Seeded RNG for reproducible world state |
| `uuid` | ^11.1.0 | Unique ID generation |
| `msgpack-lite` | ^0.1.26 | MessagePack binary serialization |
| `emoji-regex` | ^10.4.0 | Emoji detection regex |
| `normalize-wheel` | ^1.0.1 | Normalized mouse wheel events |
| `dotenv` | ^17.2.3 | Environment variable loading |
| `stats-gl` | ^3.6.0 | WebGL/WebGPU performance stats |
| `sharp` | ^0.34.5 | Image processing (PNG→WebP conversion) |
| `glob` | ^13.0.0 | File glob matching |
| `@gltf-transform/core` | ^4.2.1 | glTF model manipulation core |
| `@gltf-transform/cli` | ^4.1.0 | glTF CLI tools (etc1s, draco) |
| `@gltf-transform/extensions` | ^4.2.1 | glTF extension support |
| `@gltf-transform/functions` | ^4.2.1 | glTF transform functions |
| `vite-plugin-wasm` | ^3.5.0 | Vite WASM module support |
| `vite-plugin-top-level-await` | ^1.6.0 | Vite top-level await support |
| `vite-plugin-restart` | ^0.4.1 | Vite HMR restart on file changes |
| `vite-plugin-node-polyfills` | ^0.24.0 | Node.js polyfills for browser |
| `@vitejs/plugin-basic-ssl` | ^2.1.0 | Basic SSL for local HTTPS dev |
| `@tweakpane/plugin-essentials` | ^0.2.1 | Tweakpane essential plugins |
| `@tweakpane/plugin-camerakit` | ^0.3.0 | Tweakpane camera kit plugin |

---

## Quick Reference: Most Important Files for Customization

| Priority | File | What It Controls |
|----------|------|-----------------|
| 🔴 Critical | `sources/data/projects.js` | Your portfolio projects |
| 🔴 Critical | `sources/data/social.js` | Your social media links |
| 🔴 Critical | `sources/index.html` | Page title, meta tags, bio text, analytics |
| 🔴 Critical | `.env` | Spawn point, compression mode, analytics ID |
| 🟡 Important | `sources/data/lab.js` | Lab/experiment entries |
| 🟡 Important | `sources/data/achievements.js` | Achievement definitions |
| 🟡 Important | `sources/style/general.styl` | Color scheme, base styling |
| 🟡 Important | `package.json` | Project name, author |
| 🟢 Optional | `resources/folio-2025.blend` | 3D world geometry (needs Blender) |
| 🟢 Optional | `static/sounds/musics/` | Background music tracks |
| 🟢 Optional | `static/social/share-image.png` | Social media share preview image |
| 🟢 Optional | `static/vehicle/*.glb` | Vehicle/car 3D model |

---

## Common Gotchas & Tips

1. **`npm install --force` is required** — the `.npmrc` file sets `force = true`, but if you remove it, you'll need the `--force` flag. Several deps have peer dependency conflicts with `three@0.183.2`.
2. **WebGPU vs WebGL** — The project imports `three/webgpu` which uses TSL shaders. Most modern browsers support WebGPU (Chrome 113+, Edge 113+). Firefox and Safari may fall back to WebGL via TSL's cross-compilation.
3. **Compressed vs uncompressed assets** — During development, set `VITE_COMPRESSED=false` so your changes to GLB/texture files in `static/` take effect immediately without needing to run `npm run compress` each time.
4. **The server is optional** — Whisper messages and circuit leaderboards require a backend server that wasn't open-sourced. The offline error messages in the HTML are expected. Everything else works without the server.
5. **GitHub upstream** — This repo appears to be a fork. The upstream is `https://github.com/brunosimon/folio-2025`. Sync with upstream to get updates.
6. **Stylus is unconventional** — Most modern projects use plain CSS, Sass, or Tailwind. If you're not comfortable with Stylus (`.styl` files), you can convert the styles by running `stylus` CLI and then editing the output CSS directly.
7. **Blender file** — The `folio-2025.blend` file was authored in Blender 5.1.2 and contains a 6,378-object FRC field CAD assembly (~235 MB, Zstd-compressed). You'll need **Blender 5.x** to open it natively. Blender 4.x may also open it since both use Zstd compression.
8. **Audio formats** — The project uses various audio formats. For the best cross-browser support with Howler.js, provide both `.mp3` and `.ogg`/`.wav` versions of each sound.
