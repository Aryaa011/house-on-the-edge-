# The House on the Edge 🏚️

> *"Some doors are better left unopened."*

A fully self-contained first-person horror experience built in a single HTML file. No server, no install, no dependencies — just open in a browser and play.

---
 
## 🎮 Play
https://houseonthedge.netlify.app


---

## 🕹️ Controls

| Action | Desktop | Mobile |
|--------|---------|--------|
| Move | `W A S D` or Arrow keys | Left joystick |
| Look | Mouse (click canvas to lock) | Swipe right side |
| Sprint | `Shift` (outdoor only) | — |
| Enter house | `E` or click prompt | Tap prompt |
| Release cursor | `ESC` | — |

---

## 📖 Story

**Scene I — The Arrival**
You spawn in a dark forest outside a haunted cabin. Dead trees surround you. Flickering warm light glows from inside. Walk toward the house. When you get close enough, a glowing prompt appears — *Enter the House*.

**Scene II — Inside the House**
A decaying hospital corridor. Flickering fluorescent bulbs. Scattered furniture — a chair, a filing cabinet, an overturned gurney. Total silence for 6 seconds. Then: *"Something is coming..."* — the hooded ghost woman appears at the far end, lit by a cold spotlight. She walks toward you. The lights flicker faster as she gets closer. Her glow shifts from green to red. Your heartbeat accelerates from 52 to 140 BPM.

**Scene III — The Scare**
She reaches you. White flash. Scream. Camera shake. Black screen. Red text. A *Play Again* button appears.

**Easter Egg**
Stand completely still outdoors for 25 seconds. Something moves in the cabin window.

---

## 🏗️ Technical Architecture

### Single-file approach
All three GLB assets are base64-encoded and stored in hidden `<textarea>` elements inside the HTML. This bypasses browser JS string parser limits (which can't handle 20MB string literals) and allows the file to open directly from disk without a web server.

```
horror_full.html  (~31MB)
├── <textarea id="dHouse">     — house_environment.glb  (15MB → 19MB b64)
├── <textarea id="dCorridor">  — horror_corridor_1.glb  (5.3MB → 7MB b64)
├── <textarea id="dWoman">     — ghost_woman.glb         (3.4MB → 4.5MB b64)
├── <style>                    — all CSS inline
└── <script>                   — all game logic inline
```

### Engine
- **Three.js r128** — 3D rendering, loaded from CDN
- **GLTFLoader + DRACOLoader** — GLB model loading
- **Web Audio API** — all sound synthesised in code (no audio files)
- **Canvas API** — procedural textures (wall stains, floor tiles, ceiling panels)

### Scene graph
```
Scene 1 (outdoor)           Scene 2 (corridor)
├── outdoorModel (GLB)      ├── corridorGroup (procedural)
├── outdoorGround (plane)   │   ├── floor, ceiling, walls
├── trees (GLB children)    │   ├── doors, pipes
├── cL1, cL2 (cabin lights) │   ├── chair, gurney, cabinet
└── sun, hemi, fill, amb    ├── ghostModel (GLB animated)
                            ├── ghostLight (point, green→red)
                            ├── ghostSpot (spotlight from above)
                            └── corridorBulbs[5] (flickering)
```

### Player system
- First-person camera at eye height `1.1m`
- WASD direct position delta (no velocity lag)
- Mouse look via Pointer Lock API
- Head bob: `sin(time) * amplitude` — walking or breathing idle
- Speed: `22 units/sec` walk, `45 units/sec` sprint (outdoor only), `8 units/sec` forced in corridor
- Wall collision: AABB clamp for corridor, no raycasting needed

### Sound design (Web Audio API only)
| Sound | Method |
|-------|--------|
| Wind | Noise buffer → lowpass filter → gain ramp |
| Heartbeat | Two oscillators, 55/50 Hz, exponential decay |
| Footsteps | Short noise burst, pitch varies by surface |
| Door creak | Bandpass filtered noise |
| Drone | Sine oscillator, slow gain fade |
| Jump scare scream | Oscillator sweep 200Hz → 4000Hz in 0.25s |

### Lighting system
- **Outdoor**: Directional sun (3.5), HemisphereLight sky/ground, fill, ambient
- **Cabin**: Two PointLights with `sin(time)` flicker + random dropout
- **Corridor**: 5 independent PointLights flickering at different phases
- **Ghost**: PointLight (color shifts green→red with distance) + SpotLight from above

---

## 📁 File Structure (if using separate assets)

```
project/
├── horror_full.html                              ← single self-contained file
│
└── assets/ (source files, not needed to play)
    ├── horror_house_scene_nature_tree_and_environment.glb
    ├── horror_corridor_1.glb
    └── terrifying_hooded_horror_woman.glb
```

---

## 🧱 Build Phases Completed

| Phase | Description | Status |
|-------|-------------|--------|
| **Phase 1** | FPS engine, WASD, mouse look, house environment, spawn | ✅ Done |
| **Phase 2** | Story system, scene transitions, corridor, ghost animation | ✅ Done |
| **Phase 3** | Full atmosphere — sound, lighting, fog, flicker, jump scare | ✅ Done |
| **Phase 4** | Polish — mobile controls, loading screen, restart loop, easter egg | ✅ Done |

---

## ⚙️ Known Limitations

- **31MB file size** — takes 2–4 seconds to decode on first open (browser decodes base64 to Blob URL)
- **Corridor is procedural** — the corridor GLB asset is embedded but the rendered corridor is built from Three.js geometry due to GLB axis/scale issues in file:// context. Visually equivalent.
- **Draco CDN required** — the GLTFLoader uses `cdn.jsdelivr.net` for Draco decoders. Requires internet connection on first load.
- **File:// security** — some browsers restrict Blob URL creation from file://. If the scene doesn't load, serve via `python3 -m http.server` and open `localhost:8000`.



## 🙏 Credits

- **3D Models**: Sourced from Sketchfab
  - Horror House Scene — CGS CREATOR
  - Terrifying Hooded Horror Woman 
- **Engine**: Three.js (r128) by mrdoob and contributors


---

*Phase 1 → Phase 4 built iteratively. Each phase was a testable deliverable before moving to the next.*
