# Compression of a Twisted Hopf-Torus Lattice Manifold Using LAMMPS (BPM Beam Network)


![LAMMPS](https://img.shields.io/badge/LAMMPS-BPM%20Package-blue?style=for-the-badge)
![Model](https://img.shields.io/badge/Model-Twisted%20Hopf%20Torus%20Lattice-purple?style=for-the-badge)
![Beams](https://img.shields.io/badge/bond__style-bpm%2Frotational-orange?style=for-the-badge)
![Loading](https://img.shields.io/badge/Loading-Displacement%20Controlled-black?style=for-the-badge)
![OVITO](https://img.shields.io/badge/OVITO-Visualization-green?style=for-the-badge)
[![DOI](https://img.shields.io/badge/DOI-10.5281%2Fzenodo.24054090-blue?style=for-the-badge)](https://doi.org/10.5281/zenodo.24054090)

A **beam-network simulation of the compression of a twisted Hopf-torus lattice manifold**, using LAMMPS and the BPM package. The lattice is a torus-shaped surface sampled on a 96 x 48 grid. Every lattice edge is a breakable beam (`bond_style bpm/rotational`) and every node is a particle that carries both translation and rotation. The top slab of the torus is pushed down at constant speed while the bottom slab is held fixed, until beams begin to fail.

The simulation captures **elastic loading of the curved lattice**, **the onset and growth of beam failure**, and **the loss of load-carrying capacity**, using beams that resist stretching, shear, bending and twisting. The load on the moving slab, the reaction at the fixed slab, the number of intact beams and the kinetic energy are written every 250 steps, and the full trajectory is written for visualization in OVITO.


<img width="1600" height="1200" alt="torus" src="https://github.com/user-attachments/assets/7a85fdab-2f54-44f1-98cc-e8f7441afb8f" />

---

## Physics

The simulation models the compression of a curved lattice as a frame structure:

- Lattice nodes are particles with position and orientation; lattice edges are beams between them
- Each beam has axial, shear, twisting and bending stiffness (`bond_style bpm/rotational`)
- A beam breaks when the combined load ratio of its axial force, shear force, twisting torque and bending torque reaches 1
- The twisted torus closes with a twisted identification: the last grid column joins the first column shifted by 24 rows, so the lattice has no false seam
- Nodes that bunched together on the surface are merged (132 of 4,608 grid nodes), which keeps the shortest beams from limiting the timestep
- The top slab moves down at constant speed (`fix move`), the bottom slab stays fixed, all other nodes move freely
- The beams are the only interaction (`pair_style zero`), so there is no contact between beams that touch after failure
- Beam damping is a fraction of the critical value, so waves from the loading die out
- Optional global drag on the free nodes (`visc`), off in the recommended settings (see the section on measured force)

---

## Loading Regime

| Quantity                    | Default (`in.compress`)  | Recommended for force results |
| --------------------------- | ------------------------ | ----------------------------- |
| Final strain of loaded span | 5 %                      | 3 %                           |
| Loading time (`tload`)      | 150                      | 360                           |
| Slab speed                  | 1.21e-3                  | 3.03e-4                       |
| Strain rate                 | 3.3e-4 per time          | 8.3e-5 per time               |
| Global drag (`visc`)        | 0.002                    | 0                             |
| Steps (dt = 0.004)          | 37,500                   | 90,000                        |

All quantities are in non-dimensional (`lj`) units. A stress wave needs about 10 time units to cross the structure, and the lowest vibration periods of the free lattice are of the order of 15 to 50, so the loading must be slow compared with these times. The recommended settings keep the strain rate the same as in a short check where the load and the base reaction agreed within about 2 % (see the section on measured force). The full 90,000-step run was not tested; the time of about 3 to 4 minutes on one core is extrapolated from the default run.

---

## Model and Beam Parameters

| Parameter                 | Value                  | Role                                                          |
| ------------------------- | ---------------------- | ------------------------------------------------------------- |
| Major radius `R`          | 3.0                    | Radius of the ring (before scaling)                           |
| Minor radius `r`          | 1.1                    | Radius of the tube (before scaling)                           |
| Twist `tau`               | 1.5                    | Twist of the tube along the ring                              |
| Ripple amplitude `A`      | 0.4                    | Height of the surface ripple                                  |
| Ripple frequencies `k`, `m` | 3, 5                 | Ripples along the ring and around the tube                    |
| Grid                      | 96 x 48                | Points along the ring and around the tube                     |
| Scaling                   | largest extent = 10    | Whole lattice scaled to 10 units across                       |
| Beam radius               | 0.0266                 | Chosen so that the total beam volume is 5.0                   |
| Young's modulus `E`       | 1.0                    | Units: `lj`                                                   |
| Poisson ratio             | 0.30                   | Shear modulus G = 0.385                                       |
| Density                   | 1.0                    | Total mass 5.0, spread over the nodes                         |
| Failure stress            | 0.02                   | Sets the breaking limits of the beams                         |
| Damping ratio             | 0.3                    | Fraction of critical damping in the bond coefficients         |
| Slab thickness            | 8 % of the span        | Thickness of the fixed and loaded slabs                       |

The geometry and the bond coefficients are generated by `build_torus.py` and stored in `torus.data` and `torus.bonds`.

---

## Geometry

```
Side view (x-z plane through the torus axis, units: lj, non-dimensional):

z = +5.8 +-------------------------------------------------+  <- box top
         |                                                 |
z = +1.8 |         +-----------------------+               |  <- top of tube
         |   LOAD  |///////////////////////|  loaded slab: 1.52 < z < 1.82
z = +1.5 |   slab  +-----------------------+  moves down at speed v
         |         .    twisted tube       .               |
         |         .  (ring seen from the  .               |
         |         .   side, beams not     .               |
z = -1.5 |         +-----------------------+               |
         |   BASE  |///////////////////////|  fixed slab: -1.82 < z < -1.52
z = -1.8 |   slab  +-----------------------+               |  <- bottom of tube
         |                                                 |
z = -5.8 +-------------------------------------------------+  <- box bottom
         x = -9          ring axis x = 0           x = +9

Top view: the ring has outer diameter 10 (x from -5 to 5, y from -4.99 to 5),
and the tube is 3.63 thick (z from -1.82 to 1.82).

x, y, z: all non-periodic (boundary f f f)
```

- **Box**: 18 x 18 x 11.6 (x from -9 to 9, y from -8.994 to 9, z from -5.815 to 5.815), larger than the structure so that nothing can leave it
- **Lattice**: 96 x 48 grid on the twisted torus, 132 nodes merged, 4,476 nodes, 9,000 beams
- **Beam length**: mean 0.25, from 0.061 to 0.481
- **Loaded span**: 3.630168 (distance between the outer faces of the two slabs)
- **Slabs**: 150 nodes each, 0.2904 thick (8 % of the span)

### System Size

| Component                    | Count   |
| ---------------------------- | ------- |
| Nodes in total               | 4,476   |
| of which fixed slab (base)   | 150     |
| of which loaded slab (load)  | 150     |
| of which free nodes          | 4,176   |
| Beams                        | 9,000   |
| Nodes with 4 beams           | 4,404   |
| Nodes with 5 beams           | 48      |
| Nodes with 6 beams           | 24      |

The nodes with 5 or 6 beams are the places where bunched nodes were merged.

---

## Atom and Bond Types

| Type | Kind | Role                                                                             |
| ---- | ---- | -------------------------------------------------------------------------------- |
| 1    | Atom | Every node of the lattice (one atom type)                                        |

The 9,000 beams are split into 6 bond types of 1,500 beams each, by beam length. Each type has its own twisting and bending stiffness, because these depend on the beam length.

| Bond type | Mean length | Length range  |
| --------- | ----------- | ------------- |
| 1         | 0.136       | 0.061 - 0.173 |
| 2         | 0.188       | 0.173 - 0.207 |
| 3         | 0.227       | 0.207 - 0.246 |
| 4         | 0.267       | 0.246 - 0.289 |
| 5         | 0.313       | 0.290 - 0.334 |
| 6         | 0.370       | 0.334 - 0.481 |

---

## Simulation Phases

| Phase     | Run Steps | Simulated Time | Description                                                           |
| --------- | --------- | -------------- | --------------------------------------------------------------------- |
| Setup     | -         | -              | Data file read, bonds and groups defined, nodes at rest               |
| Loading   | 37,500    | 150            | Loaded slab moves down by 5 % of the span (0.1815), bottom slab fixed |
| **Total** | **37,500**| **150**        | Default settings, timestep 0.004                                      |

There is no equilibration step: the lattice starts stress-free, because the beams store their initial shape as the reference state when the first run starts.

---

## Expected Event Timeline

The events below were measured in one serial run with the **default settings** (`visc 0.002`). The global drag changes the forces, so timings shift if you change `visc` or the loading rate.

| Event                                       | Time  | Slab displacement | Approx. dump frame |
| ------------------------------------------- | ----- | ----------------- | ------------------ |
| First beam breaks                           | 12    | 0.015 (0.4 %)     | 3                  |
| Force on the loaded slab reaches its peak   | 30    | 0.036 (1.0 %)     | 7 to 8             |
| 1 % of the beams are broken                 | 34    | 0.041 (1.1 %)     | 8 to 9             |
| 5 % of the beams are broken                 | 67    | 0.081 (2.2 %)     | 16 to 17           |
| End of the run                              | 150   | 0.182 (5.0 %)     | 37                 |

At the end 8,299 of the 9,000 beams are intact (701 broken, 7.8 %). The number of intact beams in the header of `torus_final.data` is the quickest check that your run matches. Dump frames are every 1,000 steps (4 time units) and the last frame is written at step 37,000.

---

## Governing Equations

### Geometry of the Lattice

```
x = (R + r cos(v + tau u)) cos u
y = (R + r cos(v + tau u)) sin u
z =  r sin(v + tau u) + A sin(k u) cos(m v)

R = 3.0   r = 1.1   tau = 1.5   A = 0.4   k = 3   m = 5
u, v in [0, 2 pi), 96 values of u and 48 values of v

twisted closure : node (i, last u-column) is joined to node (i + 24 mod 48, first u-column)
                  24 = frac(tau) * 48
```

### Beam Coefficients (per bond type, L = mean beam length of the type)

```
a   = beam radius = 0.0266          A_s = pi a^2           I = pi a^4 / 4        J = pi a^4 / 2

k_radial   = E A_s                       (axial, with normalize yes)
k_shear    = 0.9 G A_s                   (shear, with normalize yes)
k_twist    = G J / L
k_bend     = E I / L

f_radial,c = s_f A_s                     f_shear,c = 0.6 s_f A_s
t_twist,c  = 0.6 s_f J / a               t_bend,c  = s_f I / a        (s_f = failure stress = 0.02)
```

For bond type 1 the file `torus.bonds` gives `k_radial = 2.221e-3`, `k_shear = 7.689e-4`, `k_twist = 2.222e-6`, `k_bend = 2.889e-6`. The axial and shear stiffness are the same for all six types.

### Breaking Criterion (LAMMPS `bpm/rotational`)

```
B = f_r / f_r,c + |f_s| / f_s,c + |tau_t| / tau_t,c + |tau_b| / tau_b,c

the beam breaks when B >= 1 (its bond type is then set to 0 and it carries no load)
```

### Loading Kinematics

```
span      = 3.630168                            (loaded span)
v         = strain * span / tload               (slab speed, default 5 % * 3.630168 / 150 = 1.21e-3)
disp(t)   = v * t = step * dt * v               (displacement of the loaded slab)
```

### Measured Force

```
F_res  = - (F_load . d)       force on the loaded slab, resisting the motion       (v_Fres, column 3)
F_base =   (F_base . d)       reaction force on the fixed slab                       (v_Fbas, column 4)
d      = (0, 0, -1)           loading direction

in a quasi-static run F_res and F_base are equal
```

---

## Group Definitions

```
loadable   <- all nodes (atom type 1)
tbase      <- nodes inside region rbase  (-1.815 < z < -1.525)
tload      <- nodes inside region rload  ( 1.525 < z <  1.815)
base       <- tbase and loadable                   (fixed, not integrated)
load       <- tload and loadable                   (moved with fix move)
free       <- all - base - load                    (integrated with fix nve/bpm/sphere)
```

---

## Output Files

| File                | Frequency          | Content                                                                 |
| ------------------- | ------------------ | ----------------------------------------------------------------------- |
| `curve_torus.dat`   | every 250 steps    | 6 columns, 151 rows (see the table below)                               |
| `torus.dump`        | every 1,000 steps  | id, type, x, y, z, c_nb (38 frames)                                     |
| `torus_final.data`  | end                | Final configuration (LAMMPS data format), broken beams are removed      |

### Columns of `curve_torus.dat`

| Column | Quantity                                                      |
| ------ | ------------------------------------------------------------- |
| 1      | Timestep                                                      |
| 2      | Displacement of the loaded slab                               |
| 3      | Force on the loaded slab resisting the motion (`v_Fres`)      |
| 4      | Reaction force at the fixed slab (`v_Fbas`)                   |
| 5      | Number of intact beams (`v_nint`)                             |
| 6      | Kinetic energy of the free nodes (`c_ke`)                     |

### Thermo Output

| Column          | Quantity                                                   |
| --------------- | ---------------------------------------------------------- |
| `step`, `time`  | Timestep and simulated time                                |
| `v_disp`        | Displacement of the loaded slab                            |
| `v_Fres`        | Force on the loaded slab, resisting the motion             |
| `v_nint`        | Number of intact beams                                     |
| `c_ke`          | Kinetic energy of the free nodes                           |

---

## Repository Structure

```
torus_lammps/
|
|-- in.compress          # LAMMPS input script (this run)
|-- torus.data           # lattice: nodes and beams
|-- torus.bonds          # beam stiffness, strength and damping (6 bond types)
|-- torus.setup          # slab regions, groups and loading direction
|-- README.md            # this file
|
|-- optional helper files/
|   |-- build_torus.py      # regenerates torus.data, torus.bonds, torus.setup
|   |-- torus.bonds_vib     # coefficients of the separate vibration run (not used here)
|   |-- to_ovito.py         # writes torus_ovito.data, a plain copy with beams for OVITO
|   |-- analyze.py          # stiffness, peak force, energy, damage from curve_torus.dat
|   |-- run_mpi.sh          # runs the case with MPI on Ubuntu or WSL
|
|-- generated on run/
    |-- curve_torus.dat
    |-- torus.dump
    |-- torus_final.data
```

---

## How to Run

### Requirements

- LAMMPS with the **BPM** and **MOLECULE** packages: <https://www.lammps.org> (the case was run with the 22 Jul 2025 release)
- An MPI runtime that matches the one LAMMPS was built with, for parallel runs
- OVITO for trajectory visualization: <https://www.ovito.org>
- Python with numpy, scipy and matplotlib, only for the optional helper scripts

Check that LAMMPS has the BPM package:

```
lmp -h | grep bpm/rotational
```

### Step 1: Choose the loading parameters

The loading is controlled from the command line, no editing is needed:

```
variable  strain  index 0.05       # final strain of the loaded span
variable  tload   index 150.0      # loading time
variable  dt0     index 0.004      # timestep
variable  visc    index 0.002      # global drag on the free nodes
variable  nout    index 250        # steps between entries of curve_torus.dat
variable  ndump   index 1000       # steps between dump frames
```

For example, the settings recommended for force results:

```
lmp -in in.compress -var visc 0 -var tload 360 -var strain 0.03
```

### Step 2: Run the simulation

Serial, from the folder that holds the files:

```
lmp -in in.compress
```

The default run takes about 1.5 minutes on one core.

With MPI on Ubuntu or WSL, with the files on `D:`:

```
cd /mnt/d/extras/lammps_manifolds/torus
mpirun.mpich -np 4 lmp -in in.compress
```

Use `mpirun.mpich` when LAMMPS comes from `pip install lammps`, because that build uses MPICH. Use `-np` up to the number of physical cores. The lattice has only 4,476 nodes, so more than about 4 ranks gives little extra speed.

If the files were edited on Windows, convert the line endings of shell scripts first: `sed -i 's/\r$//' *.sh`.

### Step 3: Visualize in OVITO

1. Open OVITO, then `File > Load File` and select `torus.data` (or `torus_ovito.data`) for the nodes and beams
2. Add the **Load trajectory** modifier and select `torus.dump`
3. Set the particle radius to about 0.08 and the bond width to about 0.04, otherwise the default sphere size hides the lattice
4. Add **Color coding** on `c_nb` and set the range by hand, start 2 and end 6. Do not adjust the range on frame 0: then nodes that lose beams fall below the range and are drawn blue like intact nodes
5. Add **Slice** (normal 0 1 0, distance 0, slab width about 1) to look inside the tube
6. Plot columns 2 and 3 of `curve_torus.dat` (force against displacement) and column 5 (intact beams)

---

## Important: the Measured Force

With the default `visc = 0.002` every free node feels a drag force, and this drag is transmitted to the loaded slab. In a 2 % compression test (same strain rate as the default) the peak force depended strongly on this setting:

| `visc`            | Peak force on the loaded slab |
| ----------------- | ----------------------------- |
| 0.002 (default)   | 2.2e-3                        |
| 0.0005            | 1.2e-3                        |
| 0.0001            | 7.4e-4                        |
| 0                 | 5.2e-4                        |

So the force values of a default run are mostly drag and not the stiffness of the lattice. The number of broken beams also depends on this setting, because the drag adds to the beam loads. For force-based results use no drag and slower loading:

```
lmp -in in.compress -var visc 0 -var tload 360 -var strain 0.03
```

At 4 times slower loading than the default and with no drag, the force on the loaded slab (column 3) and the reaction at the base (column 4) agreed within about 2 % at 1 % strain. Always compare columns 3 and 4 of `curve_torus.dat`: in a valid quasi-static run they must be close to each other before the peak. If they differ strongly, increase `tload`.

---

## What to Look for in Results

### Elastic Loading

At the start the force rises roughly linearly with the displacement and the number of intact beams stays at 9,000. The load on the slab and the reaction at the base should be nearly equal in a slow, drag-free run.

### Onset of Failure

The first beams break in the zones where the curvature of the lattice makes the bending highest. In OVITO the nodes with lost beams (`c_nb` below its initial value) show where the damage starts, and how it spreads from there.

### Damage Growth

Column 5 of `curve_torus.dat` falls in steps as beams break. In the default run 1 % of the beams are gone at 1.1 % strain, 5 % at 2.2 % strain, and 7.8 % at the end.

### Load Drop

After the peak the force falls while the slab keeps moving, because the broken beams no longer carry load. A very low force at the end means the load path through the lattice has been cut.

### Kinetic Energy

`c_ke` (column 6) must stay small compared with the stored elastic energy. A large kinetic energy means the loading is too fast.

---

## Common Errors and Fixes

| Error or symptom                                          | Cause                                                       | Fix                                                                        |
| --------------------------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------------- |
| `Cannot open input script in.compress`                    | Command run outside the folder with the files               | `cd` into the folder that holds `in.compress`                              |
| Unknown bond style `bpm/rotational`                       | LAMMPS built without the BPM package, or too old            | Install a LAMMPS that includes BPM (check with `lmp -h`)                   |
| `Cannot open file torus.data` or `torus.bonds`            | Data files not in the run folder                            | Keep `torus.data`, `torus.bonds`, `torus.setup` next to `in.compress`      |
| Several ranks print the same output, no speed-up          | `mpirun` and LAMMPS use different MPI libraries             | Use `mpirun.mpich` with the pip LAMMPS (MPICH)                             |
| `not enough slots` from mpirun (Open MPI)                 | `-np` larger than the number of physical cores              | Use fewer ranks, or add `--use-hwthread-cpus`                              |
| `Bond atoms ... missing`                                  | Communication cutoff shorter than a beam, or unstable run   | Keep `comm_modify vel yes cutoff 1.5`; lower `dt0` if the run blows up     |
| `Neighbor list overflow`                                  | Large pair cutoff with no pair forces in use                | Keep `pair_style zero 0.3`                                                 |
| Force on the loaded slab much larger than the base force  | Global drag (`visc`) or loading too fast                    | Use `-var visc 0` and a larger `tload`                                     |
| No colour change in OVITO while scrubbing frames          | Colour range was set on frame 0                             | Set the `c_nb` range by hand, start 2 and end 6                            |
| `ModuleNotFoundError: matplotlib` (helper script)         | Python without plotting libraries                           | `python3 -m pip install numpy scipy matplotlib`                            |
| `$'\r': command not found`                                | Windows line endings in a shell script                      | `sed -i 's/\r$//' run_mpi.sh`                                              |

---

## Extending the Model

| Extension                          | What to change                                                                              |
| ---------------------------------- | ------------------------------------------------------------------------------------------- |
| Denser or coarser lattice          | Set `FINE` in `build_torus.py` (1 = coarse, 2 = default, 3 = very dense), rebuild the files |
| Stronger or weaker beams           | Change the failure stress `SIGMA_F` in `build_torus.py`                                     |
| Thicker or thinner beams           | Change the total beam volume `V_SOLID` in `build_torus.py`                                  |
| Different torus shape              | Change `R`, `r`, `tau`, `A`, `k`, `m` in `build_torus.py` (keep frac(tau) x 48 an integer)  |
| Slower, drag-free loading          | Run with `-var visc 0 -var tload 360 -var strain 0.03`                                      |
| Other loading directions           | Change the slab regions and the direction in `torus.setup`                                  |
| Other lattice manifolds            | Use the same workflow for the breather, coral and radiolarian lattices                      |
| Vibration spectrum                 | Use `in.vibration` with `torus.bonds_vib` (a separate run)                                  |
| Contact between broken beams       | Add a soft repulsive pair style with `overlay/pair yes` in `bond_style bpm/rotational`      |
| Indentation or pressure loading    | Replace the moving slab by `fix indent` or a pressure wall (not implemented here)           |

---

## Citation

If you use this code in your research, please cite:

```
@software{mishra_2026_torus_lammps,
  author    = {Mishra, Akshansh},
  title     = {Compression of a Twisted Hopf-Torus Lattice Manifold
               Using LAMMPS (BPM Beam Network)},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.24054090},
  url       = {https://doi.org/10.5281/zenodo.24054090}
}
```

Plain text citation:

> Mishra, A. (2026). *Compression of a Twisted Hopf-Torus Lattice Manifold Using LAMMPS (BPM Beam Network)* [Computer software]. Zenodo. <https://doi.org/10.5281/zenodo.24054090>

---

## Author

**Akshansh Mishra**
GitHub: <https://github.com/akshansh11>

---

## License

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />
This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.

You are free to:

- **Share**: copy and redistribute the material in any medium or format
- **Adapt**: remix, transform, and build upon the material

Under the following terms:

- **Attribution**: You must give appropriate credit to Akshansh Mishra and provide a link to this repository
- **NonCommercial**: You may not use the material for commercial purposes

Copyright 2026 Akshansh Mishra. All rights reserved for commercial use.
