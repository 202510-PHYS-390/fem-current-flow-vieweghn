# KiCAD → Simulation Workflows - Complete Summary

## Quick Reference

| Your Goal | Recommended Workflow | Output |
|-----------|---------------------|---------|
| **S-parameters, impedance matching** | Gerber2EMS → OpenEMS | S11, S21, Smith chart |
| **Transmission line analysis** | Gerber2EMS → OpenEMS | Z0, propagation delay, loss |
| **RF/microwave circuits** | Gerber2EMS → OpenEMS | Field patterns, coupling |
| **Power distribution (IR drop)** | STEP → Gmsh → ElmerFEM | Voltage drop, resistance |
| **Current density analysis** | STEP → Gmsh → ElmerFEM | Current crowding, hotspots |
| **DC resistance calculation** | STEP → Gmsh → ElmerFEM | Net resistance |

---

## Three Workflows Explained

### 1. Gerber2EMS (Recommended for High-Frequency)

```
KiCAD PCB → Export Gerbers → Gerber2EMS → OpenEMS → ParaView
```

**Best for:**
- Signal integrity (>10 MHz)
- S-parameter extraction
- Impedance matching
- RF/microwave circuits
- Transmission line characterization

**Pros:**
- ✅ Automated workflow
- ✅ Uses standard manufacturing files (Gerbers)
- ✅ Well-documented
- ✅ Active development

**Cons:**
- ❌ Requires careful port placement
- ❌ Limited to high-frequency analysis
- ❌ Manual port position calculation

**Tools:** KiCAD, Gerber2EMS, OpenEMS, ParaView

**Files created:**
- `/workspace/kicad-examples/gerber2ems_template/` - Configuration templates
- `stackup.json` - Material properties
- `simulation.json` - Simulation parameters
- `fab/*.gbr` - Gerber files from KiCAD

---

### 2. STEP Import (Recommended for DC/Low-Frequency)

```
KiCAD PCB → Export STEP → Gmsh import → ElmerFEM → ParaView
```

**Best for:**
- DC current flow analysis
- Power distribution networks
- IR drop (voltage drop) calculation
- Current density visualization
- Thermal-electrical coupling

**Pros:**
- ✅ 3D geometry import
- ✅ Flexible material assignment
- ✅ Script-based (repeatable)
- ✅ Works with ElmerFEM (finite element)

**Cons:**
- ❌ Requires manual boundary condition setup
- ❌ STEP files can be complex (many surfaces)
- ❌ Steeper learning curve

**Tools:** KiCAD, Gmsh, ElmerFEM, ParaView

**Files created:**
- `/workspace/kicad-examples/step_to_elmerfem.py` - Example script
- Demonstrates full workflow from STEP → results

---

### 3. FreeCAD Macro (Optional, Not Recommended)

```
KiCAD PCB → FreeCAD macro → OpenEMS → ParaView
```

**Best for:**
- Users who prefer GUI over command-line
- Experimentation with different setups

**Pros:**
- ✅ GUI-based (easier for non-programmers)
- ✅ Visual feedback during setup

**Cons:**
- ❌ Compatibility issues (FreeCAD 1.0+ problems)
- ❌ Manual setup (slower)
- ❌ KiCAD 8 ground plane issues
- ❌ Not pre-installed (large package)

**Status:** Documented but **not recommended** - use Workflow 1 or 2 instead

**Files created:**
- `/workspace/kicad-examples/FREECAD_WORKFLOW.md` - Documentation

---

## Decision Tree

```
START: I have a KiCAD PCB design
│
├─ Need S-parameters or impedance?
│  │
│  YES → Use Workflow 1 (Gerber2EMS)
│       ├─ Export Gerber files
│       ├─ Configure stackup.json
│       ├─ Configure simulation.json
│       └─ Run: gerber2ems -a --export-field
│
└─ Need current flow or voltage drop?
   │
   YES → Use Workflow 2 (STEP Import)
        ├─ Export STEP file
        ├─ Run: python3 step_to_elmerfem.py
        ├─ Edit case.sif (boundary conditions)
        └─ Run: ElmerSolver case.sif

Optional: Want GUI?
   └─ See FREECAD_WORKFLOW.md (has compatibility issues)
```

---

## Installation Status

All tools are **pre-installed** in this dev container:

✅ KiCAD
✅ Gerber2EMS (via pipx)
✅ gerbv (Gerber viewer)
✅ Gmsh (with OpenCASCADE for STEP import)
✅ OpenEMS (FDTD simulator)
✅ ElmerFEM (FEM solver)

**Not installed** (optional):
❌ FreeCAD (compatibility issues, large package)
   - Install manually if needed: `sudo apt-get install freecad`

---

## Getting Started

### For High-Frequency Analysis (Workflow 1)

1. **Read the tutorial:**
   ```bash
   cat /workspace/kicad-examples/gerber2ems_template/README.md
   ```

2. **Design PCB in KiCAD:**
   - Add port markers on F.Fab layer
   - Set drill origin to bottom-left
   - Export Gerbers to `fab/` directory

3. **Configure simulation:**
   - Edit `stackup.json` (material properties)
   - Edit `simulation.json` (frequency, ports, mesh)

4. **Run simulation:**
   ```bash
   cd /workspace/kicad-examples/gerber2ems_template
   gerber2ems -a --export-field
   ```

5. **View results:**
   - S-parameters: `ems/simulation/s_parameters.png`
   - Fields: ParaView → Open `Et_*.vtr`

### For DC/Low-Frequency Analysis (Workflow 2)

1. **Read the example:**
   ```bash
   cat /workspace/kicad-examples/step_to_elmerfem.py
   ```

2. **Export from KiCAD:**
   - File → Export → STEP
   - Save as `pcb_example.step`

3. **Run script:**
   ```bash
   cd /workspace/kicad-examples
   python3 step_to_elmerfem.py
   ```

4. **Edit boundary conditions:**
   ```bash
   nano step_elmerfem_sim/case.sif
   # Add voltage contacts (boundary conditions)
   ```

5. **Re-run solver:**
   ```bash
   cd step_elmerfem_sim
   ElmerSolver case.sif
   ```

6. **View results:**
   - ParaView → Open `step_result*.vtu`
   - Color by: potential, current density, electric field

---

## File Structure

```
/workspace/kicad-examples/
│
├── README.md                          # Main documentation
├── KICAD_WORKFLOWS_SUMMARY.md        # This file
├── FREECAD_WORKFLOW.md               # FreeCAD workflow (optional)
│
├── gerber2ems_template/              # Workflow 1: Gerber2EMS
│   ├── README.md                     # Detailed tutorial
│   ├── stackup.json                  # Material properties template
│   ├── simulation.json               # Simulation config template
│   └── fab/                          # Place Gerber files here
│
└── step_to_elmerfem.py               # Workflow 2: STEP import example
```

---

## What Each Workflow Does

### Gerber2EMS (Workflow 1)

**Input:**
- Gerber files (manufacturing files from KiCAD)
- stackup.json (PCB layer thicknesses, materials)
- simulation.json (frequency range, ports, mesh)

**Process:**
1. Parse Gerber files to extract copper geometry
2. Generate 3D FDTD mesh based on stackup
3. Place ports at specified locations
4. Run OpenEMS time-domain simulation
5. Calculate frequency-domain S-parameters
6. Export field dumps (VTK)

**Output:**
- S-parameters (S11, S21, S12, S22)
- Smith charts
- VTK field dumps (E, H fields over time)
- Port impedances

**Typical runtime:** 5-30 minutes depending on mesh size and frequency points

---

### STEP Import (Workflow 2)

**Input:**
- STEP file (3D CAD export from KiCAD)
- Material properties (conductivity)
- Boundary conditions (voltage at contacts)

**Process:**
1. Import STEP geometry using Gmsh
2. Identify copper surfaces (by height/layer)
3. Create physical groups for materials
4. Generate FEM mesh
5. Convert mesh to ElmerFEM format
6. Solve Laplace equation (∇·(σ∇V) = 0)
7. Calculate E-field, current density, Joule heating

**Output:**
- Electric potential (voltage distribution)
- Electric field (V/m)
- Current density (A/m²)
- Joule heating (W/m³)
- Total resistance

**Typical runtime:** 1-10 minutes depending on mesh complexity

---

## Comparison Table

| Aspect | Gerber2EMS | STEP Import |
|--------|------------|-------------|
| **Physics** | FDTD (time-domain EM) | FEM (steady-state DC) |
| **Frequency** | High (>10 MHz) | DC / Low (<1 MHz) |
| **Input files** | Gerber + drill files | STEP 3D model |
| **Setup complexity** | Medium (JSON config) | High (script + manual BCs) |
| **Automation** | High (gerber2ems -a) | Medium (requires editing) |
| **Mesh type** | Cartesian (FDTD) | Unstructured (FEM) |
| **Output** | S-parameters, fields | Voltage, current, resistance |
| **Typical use** | RF design, SI | Power integrity, PDN |
| **Validation** | VNA measurements | Multimeter, IR camera |

---

## Common Pitfalls

### Gerber2EMS
1. **Port positions wrong** → measure from drill origin, not absolute coordinates
2. **Mesh too fine** → simulation takes forever, start coarse
3. **Frequency too high** → violates mesh resolution rule (λ/10)
4. **Missing gerbv** → install with `sudo apt-get install gerbv`

### STEP Import
1. **No boundary conditions** → solver runs but gives trivial result
2. **Wrong surface identification** → check z-heights to identify layers
3. **Gmsh can't open STEP** → verify OpenCASCADE support: `gmsh --version`
4. **Solver diverges** → use Direct solver (UMFPack), not iterative

---

## Validation and Verification

### For Gerber2EMS Results:
- Compare S11 with VNA measurements (if available)
- Check impedance vs. analytical formulas (microstrip, coplanar)
- Verify reciprocity: S12 should equal S21
- Sanity check: S11 + S21 ≈ 0 dB (conservation of energy, lossless)

### For STEP Import Results:
- Measure DC resistance with multimeter (compare to simulation)
- Check voltage drop matches calculated IR drop
- Verify current density is highest where trace is narrowest
- Thermal imaging (if available) should match Joule heating map

---

## Learning Resources

### Tutorials
- Gerber2EMS: https://nuclearrambo.com/wordpress/gerber2ems-a-short-tutorial-to-simulate-your-pcbs-in-openems/
- OpenEMS Examples: `/workspace/Tutorials/`
- ElmerFEM Examples: `/workspace/elmerfem-examples/`

### Reference
- OpenEMS Python API: https://docs.openems.de/python/
- ElmerFEM Tutorials: http://www.elmerfem.org/
- Gmsh Documentation: https://gmsh.info/doc/texinfo/gmsh.html

### Community
- OpenEMS Forum: https://openems.de/forum/
- KiCAD Forum: https://forum.kicad.info/

---

## Next Steps

1. ✅ **Installed:** All tools are ready in this container
2. 📖 **Read:** Start with `gerber2ems_template/README.md`
3. 🎨 **Design:** Create a simple test PCB in KiCAD
4. 🚀 **Simulate:** Run your first simulation
5. 📊 **Visualize:** Open results in ParaView
6. 🔄 **Iterate:** Improve design based on simulation results

**Remember:** Start simple! Single trace, two ports, narrow frequency range. Add complexity gradually.

---

## Support

Having issues? Check:

1. **Gerber2EMS issues:** https://github.com/antmicro/gerber2ems/issues
2. **OpenEMS issues:** https://github.com/thliebig/openEMS-Project/discussions
3. **ElmerFEM forum:** http://www.elmerfem.org/forum/

---

**Happy Simulating!** 🎉
