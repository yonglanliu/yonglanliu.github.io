---
layout: post
title: "Hands-on Practice with FEP Using OpenFE: A Step-by-Step Guide"
date: 2025-12-19
tags:
  - FEP
  - RBFE
  - free-energy
  - drug-discovery
  - computational-chemistry
  - OpenFE
description: "A practical, step-by-step tutorial for performing Relative Binding Free Energy (RBFE) calculations using OpenFE."
---

## Overview

This post presents a **hands-on, step-by-step workflow** for performing **Relative Binding Free Energy (RBFE)** calculations using **OpenFE**.  
The focus is on practical implementation, reproducibility, and methodological clarity for applications in **structure-based drug discovery**.
The fundamental of RBFE can be found in my another blog [Introduction to Alchemical Free Energy Calculation: FEP and TI for RBFE](https://yonglanliu.github.io/2025/12/19/RBFE.html)
---

The workflow covers the complete RBFE pipeline, from ligand preparation to result analysis.
**Simplified Workflow** for Ligand Processing in RBFE
1. Generate Ligand Conformations & Tautomers: Ensure you have the correct protonation states and low-energy 3D structures.
2. Charge and Parameterize Ligands: Assign force field parameters, including partial atomic charges, to every ligand.
3. Perform Atom Mapping: Use an algorithm (like Lomap or a geometric mapper) to find the MCS and the transformation path (the alchemical edge) for each pair of ligands.
4. Build the Alchemical Network: Connect all mapped pairs into a network that forms a thermodynamic cycle.
5. Setup and Run Simulations: Solvate the systems (ligand in complex, ligand in solvent) and run the molecular dynamics simulations.
---


## ⚛️ Two methods can be used for Alchemical Free Energy Calculations
1. Free Energy Perturbation (FEP)
2. Thermodynamic Integration (TI)

**OpenFE** uses FEP to calculate alchemical free energy

---

## Software and Environment

The following software stack is used in this tutorial:

- **OpenFE**
- **OpenMM**
- **RDKit**
- **Python ≥ 3.9**

## Installation: use conda or mamba
```bash
conda create -c conda-forge -n openfe_env openfe=0.0
conda activate openfe_env
```
or 
```bash
mamba create -c conda-forge -n openfe_env openfe=0.0
mamba activate openfe_env
```
---
For FEP calculation, two legs of calculation need to be performed: 
1. **Hydration leg**: ligands + water
2. **Complex leg**: ligand + target + water

## Workflow Outline

1. Ligand preparation
   * Atom mapping and ligand network construction
3. Protein–ligand complex preparation  
4. Atom mapping and ligand network construction  
5. System setup  
6. Alchemical transformations  
7. RBFE calculations  
8. Lambda scheduling  
9. Result analysis  

### 1. Ligand Preparation

Ligand preparation ensures chemical consistency across all transformations.

Key steps include:

* Protonation state assignment
* Geometry optimization
* Partial charge assignment
* Format conversion to OpenFE-compatible objects

**Load ligands**
```bash
import openfe
from rdkit import Chem

ligands_sdf = Chem.SDMolSupplier('inputs/tyk2_ligands.sdf', removeHs=False)

# Create a list of Molecules
ligand_mols = [SmallMoleculeComponent(sdf) for sdf in ligands_sdf]
```
**Chargin ligands**
```bash
from openfe.protocols.openmm_utils.omm_settings import OpenFFPartialChargeSettings
from openfe.protocols.openmm_utils.charge_generation import bulk_assign_partial_charges

charge_settings = OpenFFPartialChargeSettings(partial_charge_method="am1bcc", off_toolkit_backend="ambertools")

charged_ligands = bulk_assign_partial_charges(
    molecules=ligands,
    overwrite=False,
    method=charge_settings.partial_charge_method,
    toolkit_backend=charge_settings.off_toolkit_backend,
    generate_n_conformers=charge_settings.number_of_conformers,
    nagl_model=charge_settings.nagl_model,
    processors=1
)
```
```bash
