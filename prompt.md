# Prompt: Virtual Shooting Range — Ballistic Trajectory Visualizer

Build a single-page, offline, browser-based **exterior ballistics simulator** ("Virtual Shooting Range"). No frameworks or build step required beyond what your environment provides; no network calls at runtime (fonts may load from Google Fonts). All UI text is bilingual **Polish / English** with a language toggle; Polish is the default.

## Overall look & layout

Dark, technical, instrument-panel aesthetic — think range-finder / mil-spec software, not a consumer web app.

- Fonts: **IBM Plex Mono** (body, data) + **Chakra Petch** (section headings, uppercase, letter-spaced).
- Dark theme (default): near-black blue background `#0A0E14`, panels `#0D1219`/`#0F151D`, borders `#223146`, text `#D5DDE6`, cyan accent `#45D6FF` (soft `#8FE6FF`), plus amber `#FFB454`, green `#7EE2A8`, gold `#FFD166`, red `#FF4D4D`, violet `#B48CFF` for the bore line. Provide a full **light theme** variant (bg `#E8EDF3`, accent `#0B7EA8`, etc.) switchable at runtime; implement both as CSS custom properties on `html` / `html[data-theme="light"]`.
- Square corners everywhere (no border radius), 1px borders, thin custom scrollbars, custom slim range-slider styling (3px track, 11px round accent thumb).
- Layout: full viewport, `display:flex`:
  1. **Left (flexible): 3D range viewport** — full-bleed canvas with floating overlay panels on top (semi-transparent, backdrop-blur).
  2. **Right (fixed 332px): parameters panel** — scrollable column of control sections.

## Ballistics engine (the core)

Point-mass exterior ballistics, SI units internally:

- **Integration: RK4** with fixed small time step, simulating from muzzle until the bullet passes target distance or hits ground.
- **Drag: G1 model** — implement the standard G1 drag-coefficient table vs Mach number with interpolation; retardation scaled by ballistic coefficient (BC).
- **Air density** from temperature (°C), pressure (hPa), relative humidity (%) — compute vapor pressure (Magnus formula) and mix dry/humid air; speed of sound from temperature.
- **Wind**: speed (m/s) + direction on a **12-hour clock dial** (12 = headwind from target). Full 3D wind vector affects drag relative velocity.
- **Incline shooting**: uphill/downhill angle in degrees rotates gravity relative to line of sight.
- **Sight geometry**: scope height above bore (cm), zero distance (m) — solve the bore elevation angle so the trajectory crosses line of sight at zero distance (iterative two-pass solve). Turret corrections ELEV/WIND entered in **MOA or MIL** (unit toggle) added to the launch angles.
- Track per-sample: distance along LOS, drop vs LOS (m), horizontal drift (m), speed, time of flight. Record line-of-sight crossings, the transonic point (Mach 1.2→1.0) and impact sample at target distance. Cache the trajectory; recompute only when inputs change.

**Ammo presets** (dropdown; selecting fills MV/mass/BC/caliber, all still hand-editable):
`.22 LR 370 m/s, 40 gr, BC .138` · `9 mm 360, 124, .15` · `5.56 NATO 940, 62, .304` · `.308 Win 800, 168, .462` · `6.5 Creedmoor 823, 140, .58` · `.338 Lapua Mag 900, 250, .675`.

## Left: 3D range viewport (custom canvas renderer)

Hand-rolled 3D projection on `<canvas>` (no WebGL needed): perspective camera with orbit (drag), zoom (wheel), plus camera presets — **free orbit / behind shooter / side profile**. Draw:

- Ground plane grid, distance markers every 25/50 m with labels, horizon/sky (sky brightness slider, visual only).
- Shooting bench/shooter position at origin; target board downrange at target distance, on a post.
- **Target board sized to the selected target type** (see Targets below), with rings/zones drawn on the board in 3D and a **dimension label above the board** (e.g. "Ø 50 cm").
- **Trajectory polyline** with color grading (e.g. supersonic→transonic→subsonic), optional vertical **drop lines** from the line of sight to the trajectory, straight **line of sight** and violet **bore axis** line, impact marker, ground-impact marker if it lands short.
- Overlay panels (floating cards on the canvas):
  - **Top-left: readout "ON TARGET / NA CELU"** — drop (cm + MOA/MIL), drift, velocity + Mach, energy (J), time of flight, and **SCORE** (see Targets).
  - **Scope view inset** — circular reticle view with MIL/MOA hash reticle showing predicted impact offset.
  - **Target face inset ("TARCZA")** — 2D face-on canvas of the selected target drawn **at true scale** (rings/IPSC zones), auto-zoom to fit dispersion, crosshair, impact + ghost (no-wind) impact dots, target dimension label bottom-left and colored SCORE bottom-right.
  - **DOPE table ("TABELA POPRAWEK")** — range rows every 25 m (50 m past 400), each with drop cm & MOA/MIL, drift cm & MOA/MIL, velocity, TOF; target row highlighted; transonic row marked. Header has a small **"⎙ PDF" button**.
- Bottom bar: language toggle PL/EN, theme toggle, distance unit m/yd, MOA/MIL toggle.

## Right: parameters panel (sections, uppercase Chakra Petch headers)

Each control row: tiny uppercase label left, live value right (cyan), slim slider below. On hover, show a contextual **info box** explaining the parameter and whether it affects ballistics.

1. **AMUNICJA / AMMO** — preset dropdown; muzzle velocity; bullet mass (gr); BC (G1); caliber.
2. **KARABIN / RIFLE** — scope height; zero distance; ELEV and WIND turret corrections (MOA/MIL).
3. **ŚRODOWISKO / ENVIRONMENT** — wind speed; wind direction (clock dial widget); temperature; pressure; humidity; sky light (visual only).
4. **STRZAŁ / SHOT** — target distance; incline angle; **TARGET TYPE / RODZAJ TARCZY** select:
   - Ring target Ø 50 cm — 10 rings × 2.5 cm, score 1–10
   - Ring target Ø 100 cm — 10 rings × 5 cm
   - Steel gong Ø 30 cm — hit/miss
   - IPSC paper 45 × 57.5 cm — zones A (15×28 cm), C (30×45 cm), D (full) 
   - Custom plate — extra slider Ø 10–200 cm, hit/miss
   Scoring uses the computed impact deviation (devH, devV) at target distance; show localized result: `9/10`, `STREFA A / ZONE A`, `TRAFIONY/HIT`, `PUDŁO/MISS`.

## PDF export (DOPE)

The "⎙ PDF" button opens a new window with a **print-ready A4 page** (plain black-on-white, Courier table, Helvetica headers): title, generation timestamp, metadata block (ammo string, rifle/zero/turrets, environment, shot + target size), then the full DOPE table (range, drop cm + MOA/MIL, drift cm + MOA/MIL, velocity, energy J, TOF) with the target-distance row highlighted; footer disclaimer. Auto-invoke `window.print()` on load so the user can save as PDF.

## Tweakable props

Expose as component props/tweaks: `showDropLines` (boolean, default true) and 1–2 similar display toggles.

## Quality bar

- Everything updates live as sliders move; trajectory recompute is cached and fast.
- All numbers localized and unit-converted consistently (m/yd, MOA/MIL, cm).
- Min font ~9.5px uppercase labels, 11–12px data; generous letter-spacing on headings.
- No emoji, no rounded cards, no gradients — flat technical panels only.
- Include a short disclaimer that the simulation is educational and does not replace real zeroing.
