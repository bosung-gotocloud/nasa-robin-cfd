# 03_case

This directory contains the **HiSA simulation case** used to compute the flow around the **NASA ROBIN rotor configuration**.

The case is prepared so that the simulation can be executed directly once the mesh is copied from the mesh generation stage.

The solver used is **HiSA v0.151**.

---

# Case Structure

```
case/
 ├─ 0
 │   └─ include
 │       └─ freeStreamConditions
 ├─ constant
 └─ system
```

This is a standard **OpenFOAM case structure** containing the initial conditions, mesh configuration, and solver settings.

---

# 1. Prepare the Mesh

The mesh generated in the previous step must be copied into the case directory.

Copy the mesh from:

```
02_mesh/robinmesh/constant/polyMesh
```

to

```
03_case/case/constant/polyMesh
```

Example:

```bash
cp -r ../../02_mesh/robinmesh/constant/polyMesh case/constant/
```

---

# 2. Initial and Free-Stream Conditions

Initial conditions for the flow variables are defined in:

```
case/0
```

The **free-stream flow conditions** are defined in:

```
0/include/freeStreamConditions
```

This file specifies the freestream parameters used in the simulation, such as:

- free-stream velocity
- pressure
- density
- total energy

These parameters are included by the field files in the `0` directory.

---

# 3. AMI FaceSet Definition (topoSet)

HiSA requires **AMI patches to be defined as singleProcessorFaceSets** when running in parallel with MPI.

Therefore, the AMI interfaces are first converted to **faceSets using `topoSet`**.

The configuration is defined in:

```
system/topoSetDict
```

Example:

```bash
cat system/topoSetDict
```

```cpp
actions
(
    {   
        name AMI_faceSets1;
        type faceSet;
        action new;
        source patchToFace;
        sourceInfo
        {
            patch "(bladezone000_surface|bladezone000_surface_slave)";
        }
    }   
    {   
        name AMI_faceSets2;
        type faceSet;
        action new;
        source patchToFace;
        sourceInfo
        {
            patch "(bladezone090_surface|bladezone090_surface_slave)";
        }
    }   
    {   
        name AMI_faceSets3;
        type faceSet;
        action new;
        source patchToFace;
        sourceInfo
        {
            patch "(bladezone180_surface|bladezone180_surface_slave)";
        }
    }   
    {   
        name AMI_faceSets4;
        type faceSet;
        action new;
        source patchToFace;
        sourceInfo
        {
            patch "(bladezone270_surface|bladezone270_surface_slave)";
        }
    }   
    {   
        name AMI_faceSets5;
        type faceSet;
        action new;
        source patchToFace;
        sourceInfo
        {
            patch "(rotorzone_surface|rotorzone_surface_slave)";
        }
    }   
);
```

These face sets correspond to the **five AMI interfaces** used in the simulation.

Run:

```bash
topoSet
```

to generate the face sets.

---

# 4. Parallel Decomposition

Parallel decomposition is configured in:

```
system/decomposeParDict
```

Example:

```bash
cat system/decomposeParDict
```

```cpp
numberOfSubdomains 64;

method          scotch;

singleProcessorFaceSets 
( 
	(AMI_faceSets1 1) 
	(AMI_faceSets2 2) 
	(AMI_faceSets3 3) 
	(AMI_faceSets4 4) 
	(AMI_faceSets5 5) 
);
```

Important notes:

- **AMI faceSets must be listed in `singleProcessorFaceSets`** when running HiSA in MPI.
- Otherwise the solver will produce errors during parallel execution.

The number of processors can be changed by modifying

```
numberOfSubdomains
```

For example:

```
numberOfSubdomains 64
```

can be adjusted to match the available cores.

---

# 5. Time Integration Scheme

The time discretization scheme is defined in:

```
system/fvSchemes
```

Example:

```cpp
ddtSchemes
{
    default     bounded dualTime rPseudoDeltaT backward;
}
```

The simulation uses **dual-time stepping**.

A steady-state configuration can also be used:

```cpp
// default bounded dualTime rPseudoDeltaT steadyState;
```

---

# 6. Flow Solver and Pseudo-Time Iterations

The flow solver configuration is defined in:

```
system/fvSolution
```

Example:

```cpp
flowSolver
{
    solver            GMRES;
    GMRES
    {
        inviscidJacobian LaxFriedrichs;
        viscousJacobian  laplacian;
        preconditioner   LUSGS;

        maxIter          20;
        nKrylov          8;
        solverTolRel     1e-1 (1e-1 1e-1 1e-1) 1e-1;
    }
}
```

Pseudo-time iteration settings:

```cpp
pseudoTime
{
    nPseudoCorr     10;
	nPseudoCorrMin	8;
    pseudoTol   	1e-3 (1e-3 1e-3 1e-3) 1e-3;
	pseudoCoNum		100;
    pseudoCoNumMax  10000;
	localTimeStepping	True;
}
```

Key features:

- Linear solver: **GMRES**
- Preconditioner: **LUSGS**
- Local pseudo time stepping enabled

---

# 7. Simulation Time Settings

Simulation time parameters are defined in:

```
system/controlDict
```

Example:

```cpp
startFrom         startTime;

startTime         0;

stopAt            endTime;

endTime           0.2972; // 10 revolutions, 1 revolution : 0.02972 sec

deltaT            0.0001486;   // 200 steps per revolution

writeControl      runTime;

writeInterval     0.001486;   // every 10 steps per revolution
```

Simulation setup:

| Parameter | Value |
|---|---|
| Total revolutions | 10 |
| Time per revolution | 0.02972 s |
| Physical time steps per revolution | 200 |
| Output interval | every 10 steps |

---

# 8. Probe Locations for Experimental Comparison

Pressure and flow quantities are sampled using **point probes** defined in `controlDict`.

```cpp
pointProbes
{
    type            probes;
    libs            (sampling);
    writeControl    timeStep;
    writeInterval   1;
    fields          (p U rho rhoE);

    probeLocations
    (
        (0.053349 -0.006571 0.042693)
        (0.099028 -0.006717 0.073343)
        (0.206353 -0.003622 0.117789)
        (0.262313 -0.002507 0.132884)
        (0.476904 0.001771 0.196746)
        (0.610583 0.004535 0.206766)
        (1.008222 0.013092 0.133865)
        (1.184843 0.016971 0.084558)
        (1.371863 0.020958 0.061743)
        (1.558831 0.024948 0.03793)
        (0.892457 -0.086491 -0.127651)
        (0.901319 -0.113959 0.069097)
        (0.90245 -0.102018 0.09512)
        (0.902839 -0.073066 0.114189)
        (0.905009 0.010751 0.18934)
        (0.900927 0.080845 0.139617)
        (0.899618 0.0809 0.114652)
        (0.898216 0.107928 0.098762)
        (0.898757 0.120018 0.075751)
        (0.888696 0.098189 -0.125257)
    );
}
```

These probe locations correspond to **measurement points used for comparison with experimental data**.

---

# 9. Rotor Motion Model

Rotor motion is defined in

```
constant/dynamicMeshDict
```

using a **multibody motion model**.

For details, refer to:

- The associated **research paper**
- The multibody solver implementation:

https://github.com/bosung-gotocloud/gotocfd/tree/main/multiMotionSolver

---

# 10. Running the Simulation

After preparing the mesh:

```bash
cd case

topoSet
decomposePar

mpirun -np 64 hisa -parallel
```

Adjust the number of processors (`-np`) according to the value specified in:

```
system/decomposeParDict
```

---

The simulation computes **10 rotor revolutions** with **200 physical time steps per revolution**, and results are written every **10 time steps**.