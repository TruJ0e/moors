# MOORS — three-model chain build (Muse → GPT → Gemini, no one else)
Spec: single self-contained moors.html — Three.js (pinned CDN module), procedural Yorkshire Moors at night, first-person terrain-following walk, fully synthesized Web Audio (wind bed + spatialized eerie events). No external assets besides the Three.js module. DoD: zero console errors; real heightmap terrain; dark/foggy/eerie; pointer-lock + terrain follow; wind + >=3 distinct random spatialized sounds; >=3 detail kinds + 1 landmark; ~60fps.

## Chain protocol (Truman's order, 2026-11-21)
- Rotation: @muse → @gpt → @gemini → @muse … One full rotation = 1 round.
- PLAN phase: rounds 1–100. BUILD phase: rounds 1–100 (artifact: moors.html, full file carried in each pass, edited in place). TEST phase: rotation continues against the DoD checklist until green. FINAL: one understanding round (one pass each), then @muse delivers.
- Codex, Claude Code, Claude Web, Antigravity are OUT. Only the three named models touch this.
- Each pass: refine the carried document in place (do not append a new essay), bump the round marker, add ONE line to the change log (keep last 12), forward the whole document to the next in rotation as a bus task.

## PLAN v1 — Muse leg, round 1 (architecture baseline)
1. **Skeleton**: one HTML file; importmap-free single module import: `import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js'`. Overlay div ("click to begin") + tiny HUD (title/controls hint, fades). All JS in one module script. A global error trap surfaces any runtime error into a console-visible + on-screen flag for testability.
2. **Noise**: seeded PRNG (mulberry32) + value-noise fBm (hash-based gradient/value noise, 5 octaves) implemented in-file (~40 lines). One canonical `terrainHeight(x,z)` used by BOTH geometry displacement and player grounding — single source of truth, so walking can never disagree with the mesh.
3. **Terrain**: PlaneGeometry 600×600 m, 256×256 segments (~66k verts, fine), rotated flat, displaced by terrainHeight; vertex colors by height/slope/noise (heather purple-brown lows, sickly green mids, slate rock on steep/high) with a MeshStandardMaterial vertexColors. Distant hill ring: a second low-res ring mesh (or scaled cylinder silhouette) at ~800–1200 m, fog-faded. World edge: fog + hills hide the border; soft player clamp at ±280 m.
4. **Sky/light**: scene.background near-black blue; moon = directional light (pale blue, low intensity ~0.55) low on horizon + a moon sprite/disc mesh + faint hemisphere light (sky #16202e / ground #0a0d08, ~0.25) so shadows aren't pure void. Stars: THREE.Points ~1,500 in a dome band, sizes attenuated, slight opacity variance. Fog: THREE.FogExp2 (#0b1017, density ~0.016) for low mist feel + a few large soft alpha "mist cards" (canvas-generated radial gradient texture — procedural, allowed) drifting slowly near ground.
5. **Detail (all procedural, InstancedMesh where counts are high)**:
   - Heather/grass tufts: cross-plane tufts with canvas-generated alpha texture, InstancedMesh ~4,000, scattered by noise mask, slope-gated.
   - Dry-stone walls: several polyline runs following terrain, built from instanced rough boxes (per-instance random scale/rot/color jitter), ~600 instances, with gaps.
   - Gnarled trees: 8–12 recursive branch cylinders (tapered, dark bark color), merged geometry per tree, placed on flatter ground.
   - Rocks: instanced dodecahedrons, grey, random squash, ~250.
   - LANDMARK: standing-stone circle (9–11 tall boxes with noise-displaced verts, one fallen) on a flattened terrace + a ruined farmstead option if budget allows (stone box walls with gaps, no roof) — pick stones as primary, farmstead as stretch.
6. **Controls**: PointerLockControls-style hand-rolled (no examples import — keep single module): yaw/pitch from mousemove, WASD+arrows, delta-time velocity with smoothing/accel, speed 4.2 m/s walk (Shift 7?), head height 1.7 m, camera y = terrainHeight+1.7 with smoothed vertical follow (lerp) to avoid jitter on bumps. Collision: none needed beyond ground clamp.
7. **Audio (Web Audio, starts on the begin-click)**:
   - Wind bed: looping noise buffer (2s white noise) → bandpass filter (~400–700 Hz) → gain; two slow LFOs (0.05–0.13 Hz, incommensurate) modulate gain + filter freq for swells; a second detuned layer an octave down at lower gain.
   - Events (scheduler: every 6–18 s random, pick one, place at random azimuth 25–70 m, PannerNode HRTF, distance model inverse):
     a. Curlew cry: sine/triangle glissando down-up warble with vibrato, bandpassed, quiet.
     b. Sub drone: 45–55 Hz sine + slight beating second osc, slow attack/release, felt more than heard.
     c. Creak: short filtered sawtooth pitch-bend bursts (2–3), like a gate/branch.
     d. Far bell: struck sine partials (f, 2.76f, 5.4f) long decay, very quiet, single toll, occasionally two.
     e. Indistinct call: formant-ish bandpassed saw blips, two syllables, heavily lowpassed + distance.
   - Master: GainNode 0.9 → compressor → destination. Listener updated per frame from camera.
8. **Perf**: pixelRatio clamp 1.5; instancing for tufts/walls/rocks; merged tree geometry; no shadows except maybe one cheap moon shadow off by default (decide in plan rounds: likely NO shadow maps — fake AO via vertex color darkening in dips); target <150 draw calls.
9. **Test hooks**: `window.__moors` exposing state (player pos, terrainHeight fn, audio event log, fps meter) so headless checks can assert DoD items programmatically; a `?test` query flag that auto-starts without pointer lock for screenshots.

## Delivery (Truman's orders, 2026-11-21 voice call)
- The chain runs until the build is DONE and CHECKED (DoD green). Only then deliver.
- Final delivery: EMAIL the finished moors.html to Andrew Wade <andrew.wade@parachutesolutionsgroup.com> (found from Truman's sent mail — "Frontier Model Inventory", "Meet"), with Truman tagged (cc true.hensley.business@gmail.com).
- Organizing repo: TruJ0e/moors on GitHub (PUBLIC, his call). The three may adapt/adjust the plan as iterations require; the spec's DoD is the bar.
- Optional, his pre-approval given: also publish the game on Cloudflare as an accessible page.

## Open questions for GPT/Gemini plan passes
- Shadow strategy (none vs one 1024 directional shadow near player).
- Mist cards vs pure FogExp2 (banding/perf).
- Terrain size vs segment density tradeoff for silhouette quality.
- Event scheduler fairness (avoid repeats/clumps) and max simultaneous events.
- Farmstead as second landmark if perf allows.

CHANGE LOG: r1-claude: baseline architecture above.
