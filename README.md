# Multilateration 3D Positioning

Estimate the 3D position of a **tag** (e.g. a device or asset) from **anchor** positions and **distance** measurements (e.g. UWB range). Supports both **trilateration** (3 anchors, coplanar) and **multilateration** (4+ anchors, arbitrary geometry).

**Purpose:** Test 3D multilateration for the BareTag app.  
**Objective:** Test whether the research works with our system and in the real world by simulation in Blender.

---

## What This Project Does

Given:
- Known 3D positions of anchors (fixed reference points)
- Measured distances from each anchor to the tag

the code solves for the tag’s **(x, y, z)** position. Use cases include indoor/outdoor asset tracking, UWB-based localization, and simulation with synthetic or real geographic data.

---

## Approach

| Method | Anchors | Constraint | Use case |
|--------|--------|------------|----------|
| **Trilateration** | 3 | Anchors on the same horizontal plane (same Z) | Fewer anchors; e.g. Xu et al. 2021 style |
| **Multilateration** | 4+ | None | More anchors; better accuracy; different heights |
| **Brute-force (scipy)** | 3+ | None | General fallback; full 3D or Z-only given XY |

- **Trilateration** (`multilat_lib.trilateration`): Geometric solution using the triangle plane and Cayley–Menger determinant (see `research/A_Precise_3D_Positioning_Approach_Based_on_UWB_with_Reduced_Base_Stations.pdf`).
- **Multilateration** (`multilat_lib.multilateration_minimum_squared` / `multilateration_closed_form`): Linear least-squares / closed-form with 4+ anchors (Larsson 2022; see `research/`).
- **Brute-force** (`multilat_lib.brute_force`): Minimizes sum of squared distance errors; can solve full 3D or only Z when X,Y are fixed.

---

## Project Structure

| File | Role |
|------|------|
| **multilat_lib.py** | Core library: trilateration, multilateration, brute-force, and geo ↔ local XYZ conversion |
| **trilateration.py** | Demo: 3 coplanar anchors; reads `tag_data.csv`; uses geometric trilateration |
| **multilateration.py** | Demo: 4+ anchors (any geometry); reads `tag_data.csv`; uses brute-force solver |
| **real_geodata.py** | Demo: real lat/lon/alt anchors and tag; 2D trilateration for XY + brute-force for Z; error analysis |
| **tag_response.py** | Blender script: exports anchor/tag positions and distances from a scene to `tag_data.csv` |
| **tag_data.csv** | Input: anchor positions (X,Y,Z) and “Distance to Sphere” (tag) per row; optional “Sphere” row = ground truth |
| **research/** | Reference papers (UWB 3D positioning, Larsson distance geometry, etc.) |

---

## Requirements

- Python 3
- **numpy**, **scipy**, **sympy** (core math and optimization)
- **Blender** (optional): only for `tag_response.py` to generate `tag_data.csv` from `threeD_positioning.blend`

Install core dependencies (no Blender):

```bash
pip install numpy scipy sympy
```

For Blender export, use the project’s `requirements.txt` (includes `bpy`) in a separate env if desired.

---

## Data Format (`tag_data.csv`)

CSV with columns: **Point**, **X**, **Y**, **Z**, **Distance to Sphere**.

- Rows with `Point != "Sphere"`: anchor name and position (X,Y,Z). **Distance to Sphere** = distance from that anchor to the tag.
- Optional row with `Point == "Sphere"`: ground-truth tag position (X,Y,Z); **Distance to Sphere** can be empty.

Example:

```csv
Point,X,Y,Z,Distance to Sphere
CubeA,0.0,0.0,0.0,3.12
CubeB,-4.11,2.22,-0.01,3.81
CubeC,3.38,1.0,-0.04,4.87
Sphere,-0.87,2.24,1.99,
```

---

## How to Run

**Trilateration (3 anchors, same plane):**

```bash
python trilateration.py
```

**Multilateration (4+ anchors):**

```bash
python multilateration.py
```

**Real geodetic data (lat/lon/alt → local XYZ → estimate → error analysis):**

```bash
python real_geodata.py
```

**Generate `tag_data.csv` from Blender** (optional):

- Open `threeD_positioning.blend` in Blender and run `tag_response.py`, or use `start_project.sh` (adjust paths for your machine; script calls Blender in background then runs a demo).

---

## References

- Xu et al., *A Precise 3D Positioning Approach Based on UWB with Reduced Base Stations* (see `research/`).
- Larsson 2022, *Localization using Distance Geometry* (see `research/`).
