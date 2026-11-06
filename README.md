# 3D Conjugate Heat Transfer (CHT) Modeling and Forced Oil Cooling Analysis of DC-Link Film Capacitors

[![COMSOL Multiphysics](https://img.shields.io/badge/COMSOL-Multiphysics_6.3-blue.svg)](https://www.comsol.com/)
[![SolidWorks](https://img.shields.io/badge/CAD-SolidWorks-red.svg)](https://www.solidworks.com/)
[![Status](https://img.shields.io/badge/Status-Research_&_Simulation-success.svg)]()

Numerical investigation, literature benchmark validation, and lumped-parameter thermal reduction for the thermal management of metallized polypropylene film (BOPP) DC-link capacitors in power electronic converters.

---

## Overview

DC-link film capacitors are critical components in high-power density converters, electric vehicle traction inverters, and renewable energy conversion systems. In high-power operations, dielectric losses and Joule heating in equivalent series resistance (ESR) cause internal hotspot temperature rise, accelerating dielectric degradation and shortening operating lifespan.

While standard industrial designs predominantly rely on natural or forced air convection, elevated power densities demand enhanced cooling strategies. This repository contains the finite element simulation models, 3D CAD geometries, CFD datasets, and literature replication benchmarks investigating **forced dielectric oil cooling (5 LPM)** for a DC-link film capacitor assembly.

### Key Objectives
1. **Conjugate Heat Transfer (CHT) Modeling:** Formulate and solve 3D coupled turbulent fluid dynamics and thermal conduction across solid and fluid domains using COMSOL Multiphysics.
2. **Material Anisotropy:** Account for orthotropic thermal conductivity within the wound BOPP film capacitor core (high in-plane metallic conduction vs. low cross-layer polymer conduction).
3. **Literature Replication & Model Validation:** Benchmark and validate the thermal modeling framework against published experimental/analytical literature under natural convection across ambient temperatures ($T_a = 60^\circ\text{C}$ and $70^\circ\text{C}$) and thermal loads ($0.714\text{ W}$ and $1.0\text{ W}$).
4. **Thermal Performance Quantification:** Quantitatively assess hotspot temperature reduction achieved via 5 LPM forced dielectric oil cooling compared to natural convection.
5. **Effective Heat Transfer Coefficient Derivation ($h_{\text{eff}}$):** Extract surface-resolved convective heat transfer coefficients across exposed boundaries and determine an area-weighted, time-averaged lumped value for compact electro-thermal network models.

---

## Numerical Framework & Methodology

### 1. Multi-Physics Coupling & Governing Equations

The computational framework couples the incompressible Reynolds-Averaged Navier–Stokes (RANS) equations closed with the Realizable $k$-$\varepsilon$ turbulence model to conjugate solid-fluid thermal energy conservation equations.

#### Fluid Domain ($\Omega_f$)

Mass conservation (continuity):

$$
\nabla \cdot \mathbf{u} = 0
$$

Momentum conservation (incompressible RANS):

$$
\rho_f (\mathbf{u} \cdot \nabla)\mathbf{u} = -\nabla p + \nabla \cdot \left[ (\mu_f + \mu_T)\left(\nabla \mathbf{u} + (\nabla \mathbf{u})^T\right) \right] + \rho_f \mathbf{g}
$$

Thermal energy transport:

$$
\rho_f C_{p,f} (\mathbf{u} \cdot \nabla T) = \nabla \cdot \left[ (k_f + k_T)\nabla T \right]
$$

where turbulent eddy viscosity $\mu_T$ and turbulent thermal conductivity $k_T$ are evaluated through the Realizable $k$-$\varepsilon$ closure and Reynolds analogy ($Pr_T = 0.85$).

#### Solid Domains ($\Omega_s$)

Transient thermal conduction with volumetric heat dissipation:

$$
\rho_s C_{p,s} \frac{\partial T}{\partial t} = \nabla \cdot (\mathbf{k}_s \cdot \nabla T) + \dot{q}_v
$$

For the active wound capacitor core ($\Omega_{\text{core}}$), an orthotropic thermal conductivity tensor is applied:

$$
\mathbf{k}_{\text{core}} = \begin{bmatrix} k_{xx} & 0 & 0 \\ 0 & k_{yy} & 0 \\ 0 & 0 & k_{zz} \end{bmatrix} = \begin{bmatrix} 1.1362 & 0 & 0 \\ 0 & 0.2200 & 0 \\ 0 & 0 & 0.2200 \end{bmatrix} \quad \left[\text{W}/(\text{m}\cdot\text{K})\right]
$$

This formulation captures high in-plane thermal conduction parallel to the continuous metallization layers ($k_{xx}$) versus cross-layer contact resistance across stacked polymer films ($k_{yy}, k_{zz}$).

#### Domain Symmetry & Boundary Conditions

* **Longitudinal Center Plane ($\Gamma_{\text{sym}}$):** Half-symmetry domain halves the computational degrees of freedom (DOF) while permitting dense boundary layer resolution.
* **Solid-Fluid Wetting Interface ($\Gamma_{\text{int}}$):** Strict Dirichlet–Neumann heat flux continuity ($T_s = T_f$ and $-n \cdot (k_s \nabla T_s) = -n \cdot ((k_f + k_T)\nabla T_f)$).
* **External Case Boundaries ($\Gamma_{\text{ext}}$):** Mixed boundary condition accounting for natural convection ($h_{\text{nat}} = 9.49 \text{ W}/(\text{m}^2\cdot\text{K})$) and thermal radiation ($\varepsilon_{\text{rad}} = 0.85$).

### 2. Numerical Discretization & Solver Architecture

* **Numerical Stabilization:** Streamline Upwind Petrov–Galerkin (SUPG) and Pressure-Stabilizing Petrov–Galerkin (PSPG) formulations.
* **Segregated Solver Scheme:**
  * **Step 1 (Hydrodynamics):** Solves velocity, pressure, $k$, and $\varepsilon$ using GMRES with multigrid preconditioning.
  * **Step 2 (Energy Conservation):** Solves the conjugate temperature field $T$ across all solids and fluid domains using the direct PARDISO solver.

---

## Thermophysical Material Properties

| Domain | Component / Material | Density $\rho$ [kg/m³] | Heat Capacity $C_p$ [J/(kg·K)] | Thermal Conductivity $k$ [W/(m·K)] | Dynamic Viscosity $\mu$ [Pa·s] |
| :--- | :--- | :---: | :---: | :---: | :---: |
| $\Omega_{\text{core}}$ | Wound BOPP Core | 907.17 | 1888.14 | $k_{xx}=1.1362$, $k_{yy,zz}=0.2200$ | — (Solid) |
| $\Omega_{\text{epoxy}}$ | Potting Epoxy Resin | 1820.00 | 1340.00 | 0.1500 (isotropic) | — (Solid) |
| $\Omega_{\text{PPS}}$ | Capacitor Housing (PPS) | 1650.00 | 1000.00 | 15.0000 (isotropic) | — (Solid) |
| $\Omega_{\text{Al}}$ | Aluminum Enclosure | 2700.00 | 900.00 | 238.0000 (isotropic) | — (Solid) |
| $\Omega_f$ | Dielectric Coolant Oil | 880.00 | 1900.00 | 0.1300 | 0.0250 (25 cP) |

* **Natural Convection & Radiation (Outer Air Boundaries):** $h_{\text{nat}} = 9.49 \text{ W}/(\text{m}^2\cdot\text{K})$, surface emissivity $\varepsilon_{\text{rad}} = 0.85$.
* **Heat Dissipation:** Nominal rated thermal loss $P_{\text{rated}} = 0.714\text{ W}$ ($P_0 = 0.357\text{ W}$ in half-symmetry); comparative load evaluated at $P = 1.0\text{ W}$.

---

## Results Summary

### 1. Thermal Comparison: Natural Air Convection vs. Forced Oil Cooling ($T_a = 40^\circ\text{C}$, $P = 0.714\text{ W}$)

| Cooling Configuration | Ambient Temp ($T_a$) | Heat Loss ($P$) | Steady-State Hotspot ($T_{\text{max}}$) | Temperature Rise ($\Delta T$) | Surface Temp ($T_{\text{surface}}$) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Natural Convection (Air)** | 40.0 °C | 0.714 W | **47.63 °C** | 7.63 °C | 43.49 °C |
| **Forced Dielectric Oil (5 LPM)** | 40.0 °C | 0.714 W | **43.44 °C** | **3.44 °C** | 40.39 °C |

> Under 5 LPM forced dielectric oil cooling, the capacitor core temperature rise is reduced from **7.63 °C down to 3.44 °C** (a reduction of **54.9%** in temperature rise compared to natural air cooling).

### 2. Literature Benchmark Replication (Natural Convection)

The simulation methodology was benchmarked against reference published data under natural cooling across multiple ambient temperatures and heat dissipation rates:
* **$T_a = 60^\circ\text{C}, P = 0.714\text{ W}$:** Core hotspot reaches $67.30^\circ\text{C}$ ($\Delta T = 7.30^\circ\text{C}$); casing reaches $63.23^\circ\text{C}$.
* **$T_a = 70^\circ\text{C}, P = 0.714\text{ W}$:** Core hotspot reaches $77.14^\circ\text{C}$ ($\Delta T = 7.14^\circ\text{C}$); casing reaches $73.11^\circ\text{C}$.
* **$T_a = 60^\circ\text{C}, P = 1.000\text{ W}$:** Core hotspot reaches $70.21^\circ\text{C}$ ($\Delta T = 10.21^\circ\text{C}$); casing reaches $64.52^\circ\text{C}$.
* **$T_a = 70^\circ\text{C}, P = 1.000\text{ W}$:** Core hotspot reaches $79.98^\circ\text{C}$ ($\Delta T = 9.98^\circ\text{C}$); casing reaches $74.34^\circ\text{C}$.

### 3. Effective Convective Heat Transfer Coefficient ($h_{\text{eff}}$)

To bridge 3D CFD field data with computationally efficient lumped-parameter thermal models, time-averaged trapezoidal numerical integration was applied to the five wetting surfaces:
* **Surface 1:** $\bar{h}_1 = 107.22 \text{ W}/(\text{m}^2\cdot\text{K})$ ($A_1 = 1995\text{ mm}^2$)
* **Surface 2 (Flow-facing face):** $\bar{h}_2 = 426.60 \text{ W}/(\text{m}^2\cdot\text{K})$ ($A_2 = 1425\text{ mm}^2$)
* **Surface 3:** $\bar{h}_3 = 114.71 \text{ W}/(\text{m}^2\cdot\text{K})$ ($A_3 = 875\text{ mm}^2$)
* **Surface 4:** $\bar{h}_4 = 116.13 \text{ W}/(\text{m}^2\cdot\text{K})$ ($A_4 = 1425\text{ mm}^2$)
* **Surface 5:** $\bar{h}_5 = 113.94 \text{ W}/(\text{m}^2\cdot\text{K})$ ($A_5 = 875\text{ mm}^2$)

Applying area-weighted spatial averaging across the total surface area ($A_{\text{total}} = 6595\text{ mm}^2$):

$$
h_{\text{eff}} = \frac{\sum_{i=1}^5 \bar{h}_i A_i}{\sum_{i=1}^5 A_i} = 180.04 \text{ W}/(\text{m}^2\cdot\text{K})
$$

In comparison to natural air convection ($h_{\text{nat}} \approx 9.49 \text{ W}/(\text{m}^2\cdot\text{K})$), 5 LPM dielectric oil circulation provides an approximately **19-fold increase** in effective convective heat removal capability.

---

## Visuals & Geometries

### CAD Model & Computational Assembly
| Single Capacitor Geometry | Cross-Sectional Internal View | 3D CFD Computational Setup |
| :---: | :---: | :---: |
| <img src="Images/CADS/Single%20Capacitor.PNG" width="260"/> | <img src="Images/CADS/Single%20Capacitor%20Sectional%20View%201.PNG" width="260"/> | <img src="Images/CADS/CAD%20Model%20for%20CFD.png" width="260"/> |

### Thermal Field & Validation
| 5 LPM Oil-Cooled Temperature Field ($T_a = 40^\circ\text{C}$) | Natural Convection Replication Curve ($T_a = 60^\circ\text{C}, 1\text{ W}$) |
| :---: | :---: |
| <img src="Images/CFD%20Images/5%20LPM%20Oil%20Cooled%20Amb%2040%20degree.png" width="400"/> | <img src="Images/Replication%20Images/Ta%2060%201%20watt-%20Graph.png" width="400"/> |

---

## Repository Structure

```text
.
├── CADS 1.2/                     # SolidWorks part and assembly CAD files & STEP neutral geometries
│   ├── Assem1.STEP
│   ├── Geometry with better symmetry.STEP   # Symmetrical half-domain used in CFD
│   ├── Recovered Symmetry Step.STEP
│   ├── Entire Assembly.SLDASM
│   ├── Single Capacitor.SLDASM
│   └── *.SLDPRT                  # Core, fluid channel, housing shell, resin, pins, cover
│
├── Data/
│   ├── CFD Data/
│   │   ├── Temperature Graph - 5LPM- Ta=40 Degree.csv   # Transient temperature profile (oil cooling)
│   │   └── h of 5 surfaces.csv                          # Surface-resolved convective coefficients
│   └── Replication/
│       ├── Exact Replication Ta=60.csv                  # Baseline benchmark at 60 °C (0.714 W)
│       ├── Exact Replication Ta=70.csv                  # Baseline benchmark at 70 °C (0.714 W)
│       ├── For Comparison with CFD Ta 40 and .714 watt.csv
│       ├── Replication with 1 watt Ta=60.csv            # Benchmark at 60 °C (1.0 W)
│       └── Replication with 1 watt Ta=70.csv            # Benchmark at 70 °C (1.0 W)
│
├── Images/
│   ├── CADS/                     # CAD renderings and sectional views
│   ├── CFD Images/               # 3D temperature contour distributions
│   └── Replication Images/       # Transient temperature response curves and comparison charts
│
├── Models/
│   ├── Oil Cooling of DC link Capacitor.mph    # Primary 3D CHT CFD model in COMSOL Multiphysics 6.3
│   ├── Replication from the paper.mph          # Replication model of reference literature
│   ├── Old Model.mph                           # Historical baseline model
│   └── Old Model 2.mph                         # Intermediate iteration
│
├── Paper Writing Contents/       # Technical documentation & manuscript derivations
│   ├── average h.pdf                           # Mathematical derivation of lumped h_eff
│   ├── cfd_setup_methodology.pdf               # Formulation of governing equations & solver setup
│   └── simulation_parameters (1).pdf           # Material matrix & boundary condition reference
│
├── Papers/                       # Relevant reference literature on DC-link capacitor thermal modeling
│
├── .gitignore                    # CAD, COMSOL, and OS-specific ignore rules
└── README.md                     # Research overview and documentation
```

---

## Software Requirements

* **COMSOL Multiphysics 6.3**
  * CFD Module (Turbulence: Realizable $k$-$\varepsilon$)
  * Heat Transfer Module (Conjugate Heat Transfer, Surface-to-Ambient Radiation)
* **Dassault Systèmes SolidWorks** (for CAD modifications via `.SLDASM` / `.SLDPRT`) or any CAD viewer supporting `.STEP`
* **Python 3.x** (for CSV data post-processing and plotting)
