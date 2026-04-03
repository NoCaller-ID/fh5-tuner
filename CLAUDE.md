# CLAUDE.md — fh5-tuner

This file provides context for AI assistants working in this repository.

## Project Overview

`fh5-tuner` is a single-page web application for generating detailed car tuning setups in Forza Horizon 5. It calculates handling, gearing, alignment, tire pressure, differential, and aero settings based on vehicle specs and dynamometer data entered by the user.

## Repository Structure

```
fh5-tuner/
├── index.html      # Entire application (React, CSS, JS — ~600 lines)
└── README.md       # Placeholder
```

There is no build system, no package.json, no transpiler, and no test suite. The entire application is self-contained in `index.html`.

## Technology Stack

- **React 18.2.0** loaded from CDN (unpkg)
- **Pure `React.createElement()` calls** — no JSX, no Babel
- **Inline CSS-in-JS** for all styles, plus a small `<style>` block for badge classes and global resets
- **Browser `localStorage`** for persisting saved tunes (max 50)
- **Google Fonts CDN** for Outfit and Space Mono typefaces

## Running the App

Open `index.html` directly in any modern browser. No server, no build step, no dependencies to install.

## Architecture

### Single-File Layout (index.html)

1. `<head>` — font imports, root CSS variables, badge/body styles
2. `<script>` — all JavaScript in one block:
   - Constants and lookup tables
   - Calculation functions
   - React component definitions
   - `ReactDOM.createRoot(…).render(…)` call
3. `<body>` — single `<div id="root">` mount point

### Key Constants

| Constant | Purpose |
|----------|---------|
| `DRIVETRAINS` | `["RWD","FWD","AWD"]` |
| `ENGINE_POS` | `["Front","Mid","Rear"]` |
| `TIRES` | `["Street","Sport","Race","Slicks"]` |
| `GEAR_COUNTS` | `[4,5,6,7,8]` |
| `TIRE_K` | `0.0744` — used in speed/gearing math |
| `HIGH_PTW` | `0.28` — power-to-weight threshold for archetype selection |
| `C` | Color palette object (20+ named colors) |
| `GP` | Gradient preset object (pink, flat, gold, neon, blue) |
| `ARCHETYPES` | 18 baseline configs keyed by `enginePos-drivetrain-tier` |
| `GEAR_RANGES` | Min/max ratio bounds per gear (1–8) and final drive |

### Core Calculation Functions

| Function | Signature | Purpose |
|----------|-----------|---------|
| `calcChassis` | `(inp, eOff, xOff)` | Produces all handling values: springs, ARBs, damping, camber, toe, caster, ride height, tire pressure, brakes, diff, aero |
| `calcGearing` | `(dyno, nG, tgtSpd, dt)` | Computes gear ratios, final drive, shift RPMs, launch RPM, top speed |
| `getArchetype` | `(enginePos, dt, ptw)` | Returns baseline camber and pressure for the vehicle configuration |
| `scaleCamberForCompound` | `(base, tc)` | Adjusts camber by tire compound |
| `scalePressureForCompound` | `(base, tc)` | Adjusts pressure by tire compound |
| `recalcForTune` | `(tune)` | Recalculates outputs for a saved tune using current range settings |
| `getTuneOutputs` | `(tune)` | Extracts the stored tuning values from a saved tune |
| `computeTrends` | `(tunes)` | Aggregate statistics across the tune library |
| `loadTunes` / `saveTunes` | — | Read/write tune library to localStorage |

### State (managed in `App()`)

```
inp          Vehicle inputs (hp, weight, weightDist, drivetrain, enginePos, tireCompound, tireWidthF, tireWidthR, hasAero, inputMethod)
dyno         Power curve data (peakHp, peakHpRpm, peakTq, peakTqRpm, redline, tgtMph, climbRpm, falloffRpm)
nG           Number of gears (4–8)
tgtSpd       Target top speed (mph)
ranges       Min/max bounds for springs, ARBs, rebound, bump
eOff         Entry balance offset (−10 to +10)
xOff         Exit balance offset (−10 to +10)
tunes        Array of saved tune objects
showTunes    Boolean — tune library panel visibility
showSave     Boolean — save-tune form visibility
showTrends   Boolean — trends panel visibility
loadedTuneId ID of currently loaded tune
expandedTuneId ID of tune with expanded detail view
```

Computed values use `useMemo`:
- `ch` — output of `calcChassis(inp, eOff, xOff)`
- `ge` — output of `calcGearing(dyno, nG, tgtSpd, inp.drivetrain)`

### React Components

| Component | Purpose |
|-----------|---------|
| `App` | Root — owns all state, renders every panel |
| `Sec` | Section header (title + optional subtitle) |
| `Row` | Label + value display row, supports badge |
| `NumIn` | Number input with min/max clamping and validation |
| `TxtIn` | Text input for names/notes |
| `Pill` | Single-select toggle button (used for drivetrain, engine pos, etc.) |
| `Toggle` | Boolean checkbox-style control |
| `WtSlider` | Weight distribution slider (F% / R%) |
| `RangeCalc` | Slider that maps a % position to a value within a custom range |
| `GearPos` | Spectrum bar showing a gear ratio's position within its valid range |
| `BalRow` | Balance offset slider (−10 to +10) |
| `PowerBar` | SVG power curve with TQ/HP markers |
| `ArchTag` | Badge showing active archetype (tested/theory status) |
| `DeltaVal` | Inline diff display between saved and current value |
| `TuneDetail` | Expanded view: notes, full parameter comparison, delete button |
| `TrendsPanel` | Aggregate statistics over the saved tune library |

### Saved Tune Data Model

```javascript
{
  id: <timestamp>,
  name: "string",
  car: "string",
  date: "M/D/YYYY",
  notes: "string",
  // All input state snapshotted at save time:
  inp: { hp, weight, weightDist, drivetrain, enginePos, tireCompound, tireWidthF, tireWidthR, hasAero, inputMethod },
  dyno: { peakHp, peakHpRpm, peakTq, peakTqRpm, redline, tgtMph, climbRpm, falloffRpm },
  ranges: { sprMin, sprMax, arbMin, arbMax, rebMin, rebMax, bmpMin, bmpMax },
  eOff: <number>,
  xOff: <number>
}
```

## UI Conventions

- **Dark theme**: Near-black backgrounds (`#0a0608`, `#120d10`), pink/gold accents
- **Badge types**: `.badge-tested` (green — in-game validated), `.badge-theory` (gold — theoretical), `.badge-fh5` (blue — FH5 info)
- **Layout**: Mobile-first, max-width 540px cards, CSS safe-area insets for notched devices
- **No JSX**: All UI built with `React.createElement(tag, props, ...children)` or the shorthand `e(tag, props, ...children)` where `e = React.createElement`

## Development Conventions

- **No build step**: Edit `index.html` directly and reload in browser to test
- **No linter or formatter**: No config files exist — format code consistently with the surrounding style (2-space indentation, single quotes)
- **No test suite**: Validate changes by manually testing in the browser
- **Keep it single-file**: Do not split into multiple files or introduce a bundler unless explicitly requested
- **Avoid CDN changes**: The React/ReactDOM CDN URLs are pinned to 18.2.0 — do not upgrade without reason
- **localStorage key**: `fh5_tunes` — do not rename this without a migration path, as it would break existing saved data
- **Max tune cap**: 50 tunes in localStorage — enforced in the save handler
- **Version comments**: New features are marked with version comments like `// V9` or `// V9.2` — continue this convention

## Archetype System

Archetypes map a vehicle configuration to baseline camber and pressure values. The key format is:

```
"<enginePos>-<drivetrain>-<tier>"
// e.g. "Front-RWD-High", "Mid-AWD-Low", "Rear-RWD-High"
```

Tier is determined by PTW (power-to-weight): PTW ≥ `HIGH_PTW` → "High", else "Low".  
Each archetype entry has a `tested` boolean indicating whether the values have been validated in-game. Untested archetypes show a `.badge-theory` indicator.

When modifying archetypes, update both the `ARCHETYPES` constant and the `ArchTag` component if display changes are needed.

## Common Tasks

### Adding a new tuning parameter
1. Add the calculated value to the return object of `calcChassis()`
2. Display it with a `Row` component in the relevant section of `App`
3. If it requires a user-adjustable range, add range state to `inp`/`ranges` and a `RangeCalc` component

### Adding a new vehicle input
1. Add the field to the `inp` state default object in `App`
2. Add the corresponding UI control (usually `NumIn` or `Pill`) in the Vehicle Inputs section
3. Use the new value in `calcChassis()` or `calcGearing()` as appropriate
4. Ensure it is included in the saved tune data model so tunes remain complete

### Modifying gearing math
The gearing calculator is in `calcGearing()`. It uses `GEAR_RANGES` to clamp ratios and `TIRE_K` for speed calculations. Shift points are computed as a ratio of `peakHpRpm` to maintain power band through shifts.

### Adding a saved tune field
1. Add to the object created in the save handler inside `App`
2. Add to the `TuneDetail` component display
3. Consider backwards compatibility — old tunes in localStorage won't have the new field, so default to a safe fallback when reading
