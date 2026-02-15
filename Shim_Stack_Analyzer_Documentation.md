# Shim Stack Analyzer — Technical Documentation

## Overview

The **Shim Stack Analyzer** is a browser-based engineering tool that simulates the structural behaviour of a cantilever beam composed of stacked shims (thin metal plates). It calculates and visualises deflection, bending moment, shear force, and bending stress under a uniformly distributed load.

**Live URL:** [https://milla421.github.io/beam_app/](https://milla421.github.io/beam_app/)

---

## What It Does

| Feature | Description |
|---|---|
| **Beam Visualisation** | Animated beam that bends under load with colour-coded shim layers |
| **Deflection Graph** | Shows the deformed shape (X vs Y displacement) |
| **Bending Moment Graph** | Displays moment distribution along the beam |
| **Shear Force Graph** | Shows shear force distribution along the beam |
| **Live Math Sheet** | Real-time equations panel showing every step of the calculation with actual numerical values |
| **Shareable Links** | Configuration can be shared via URL parameters |

---

## How It Works

### Input Parameters

Users configure the following via the sidebar:

- **Elastic Modulus (E):** Material stiffness in GPa (default: 200 GPa for steel)
- **Width (b):** Shim width or diameter in mm
- **Distributed Load (w):** Applied uniform load in N/mm, controlled via slider
- **Shim Stack:** Each shim has a length (mm) and thickness (mm). Shims are automatically sorted longest-first. Multiple shims can be added or removed.

### Physics Model

The tool models a **cantilever beam** (fixed at one end, free at the other) loaded with a uniformly distributed load:

```
    w (N/mm) ↓↓↓↓↓↓↓↓↓↓↓↓
    ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
    █ ← Fixed wall (root)     Free tip →
```

**Unbonded shim stack:** The shims are free to slide relative to each other (they are not glued/bonded). This means each shim bends independently, and the total stiffness is calculated as:

```
EI_total = Σ E × (b × tᵢ³ / 12)
```

where `tᵢ` is the thickness of each shim. This is different from a bonded stack, which would require the parallel axis theorem and produce a much stiffer beam.

### Numerical Method

The solver uses **arc-length integration**, a geometrically nonlinear method that accurately handles large deflections where the beam curves significantly:

1. **Shear Force** — Integrated backward from the free tip: `V(x) = w × (L − x)`
2. **Bending Moment** — Integrated using the trapezoidal rule: `M(x) = −w × (L − x)² / 2`
3. **Curvature** — Computed from moment and stiffness: `κ = M / EI`
4. **Angle & Position** — Forward-integrated using arc-length coordinates:
   - `θ(s) = ∫ κ(s) ds`
   - `X(s) = ∫ cos(θ) ds`
   - `Y(s) = ∫ sin(θ) ds`

Integration uses 500 grid points with the **trapezoidal rule** for second-order accuracy.

### Boundary Conditions

| Location | Condition |
|---|---|
| Root (wall, x = 0) | θ = 0, X = 0, Y = 0 (fixed) |
| Tip (free end, x = L) | V = 0, M = 0 (no force/moment) |

### Key Equations Summary

| Quantity | Formula | Units |
|---|---|---|
| Second moment of area | I = b × t³ / 12 | mm⁴ |
| Flexural rigidity | EI = E × I | N·mm² |
| Max shear force | V_max = w × L | N |
| Max bending moment | M_max = w × L² / 2 | N·mm |
| Tip deflection (analytical) | δ = w × L⁴ / (8 × EI) | mm |
| Max bending stress | σ = E × |κ| × (t/2) | MPa |

### Live Calculation Sheet

The website includes a **Live Calculation Sheet** panel that updates in real time as the user adjusts parameters. It displays:

1. All input parameter values
2. Per-shim stiffness calculations (I and EI for each shim) in a table
3. Shear force formula with computed maximum value
4. Bending moment formula with computed maximum value
5. Tip deflection — both the analytical (small-deflection) formula and the numerical solver result
6. Bending stress at the extreme fibre of the thickest shim
7. A summary of the numerical integration method with step-by-step equations

This allows any user to **follow along with every calculation** and verify accuracy independently.

---

## Technology

- **Single-file HTML application** — no frameworks, no build step, no dependencies
- **Vanilla JavaScript** — all physics and rendering computed client-side
- **HTML5 Canvas** — hardware-accelerated 2D rendering with HiDPI (Retina) support
- **GitHub Pages** — hosted as a static site, live 24/7 with no server costs

---

## Validation

The solver has been validated against closed-form analytical solutions for a single shim under small deflections:

| Test Case | Analytical | Numerical | Agreement |
|---|---|---|---|
| V_max = w × L | Exact match | Exact match | ✓ |
| M_max = w × L² / 2 | Exact match | Exact match | ✓ |
| δ_tip (small load) | Close match | Close match | ✓ |
| σ_max = M × t/(2I) | Close match | Close match | ✓ |

For large deflections, the numerical solver diverges from the linear analytical solution — this is expected and correct, as the arc-length method captures geometric nonlinearity that the linear formula cannot.

---

## How to Use

1. Open [https://milla421.github.io/beam_app/](https://milla421.github.io/beam_app/)
2. Set material properties (Elastic Modulus, Width)
3. Add/remove shims and set their lengths and thicknesses
4. Adjust the load slider to see the beam deflect in real time
5. View the displacement, moment, and shear graphs
6. Scroll down to the **Live Calculation Sheet** to see all formulas with actual values
7. Use **Copy Shareable Link** to share a specific configuration
