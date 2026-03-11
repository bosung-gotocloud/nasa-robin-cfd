# 01_geometry

This directory contains the **STEP geometry files** used for the ROBIN rotor CFD model.  
The geometries include the rotor blade, fuselage, and pylon used to construct the computational mesh.

All geometries are provided in **STEP format** to ensure compatibility with CAD software and mesh generation tools such as **Salome**.

---

# Geometry Files

| File | Description |
|-----|-------------|
| `naca0012-0.86R.stp` | Baseline rotor blade geometry based on the NACA0012 airfoil with rotor radius **0.86 m** and **−8° twist** |
| `blade000_NACA0012-twist_-8_coning1.7.step` | Rotor blade geometry at **azimuth angle 0°** with **−8° twist**, **1.7° coning**, and **24% root cut** |
| `robin_fuselage.step` | ROBIN fuselage geometry generated using **stepROBIN** |
| `robin_pylon.step` | ROBIN pylon geometry generated using **stepROBIN** |

---

# Rotor Blade Geometry

## Baseline Blade

### `naca0012-0.86R.stp`

This file defines the **baseline rotor blade geometry** used in the model.

Blade characteristics:

- Airfoil: **NACA0012**
- Rotor radius: **0.86 m**
- Blade twist: **−8°**

This geometry represents the **twisted rotor blade without coning or azimuthal positioning**.

---

## Azimuth Blade Geometry

### `blade000_NACA0012-twist_-8_coning1.7.step`

This file represents the **rotor blade positioned at azimuth angle 0°** with coning and root cut applied.

Blade parameters:

- Airfoil: **NACA0012**
- Twist: **−8°**
- Coning angle: **1.7°**
- Root cut: **24%**
- Azimuth angle: **0°**

The blade transformation is defined with respect to the **rotor center**

```
(0.696 0.051 0.322)
```

This geometry is used to generate the **blade000 mesh region**.

---

# Fuselage and Pylon Geometry

## `robin_fuselage.step`

STEP geometry of the **ROBIN fuselage**.

---

## `robin_pylon.step`

STEP geometry of the **ROBIN rotor pylon**.

Both geometries are generated using the **stepROBIN** repository:

https://github.com/bosung-gotocloud/gotocfd/tree/main/stepROBIN

The repository provides scripts used to construct the **parametric ROBIN fuselage and pylon geometries**.

---

# Notes

- All geometries are stored in **STEP format** for interoperability with CAD and mesh generation tools.
- These geometries are used in the next stage of the workflow to generate meshes in **Salome**.
- The resulting meshes are exported to **OpenFOAM format** and merged into a single computational mesh.
