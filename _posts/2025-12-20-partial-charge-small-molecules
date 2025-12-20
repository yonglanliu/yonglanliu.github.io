---
layout: post
title: "Partial charge assignment for small molecules"
date: 2025-12-20
tags:
 - RBFE
 - MD simulations
 - FEP
 - molecular docking
 - virtual screening
 - free-energy
 - drug-discovery
 - computational-chemistry
---

### Partial charge assignment for small molecules
Partial atomic charges are one of the most deceptively simple — and critically important — components of small-molecule modeling. 
Whether you are running virtual screening, molecular docking, classical MD, or relative binding free energy (RBFE/FEP) calculations, 
the choice of charge model directly affects electrostatics, hydrogen bonding, solvation, and ultimately your predicted affinities.

This post provides a practical, workflow-oriented overview of partial charge assignment for small molecules, with concrete recommendations 
for **docking vs. free-energy calculations**, and explains how popular tools such as **RDKit** and **[AmberTools](https://ambermd.org/AmberTools.php)** fit into modern pipelines.

### Why partial charges matter
In classical force fields, electrostatic interactions are modeled using fixed partial charges assigned to atoms. These charges are meant to represent the average electron distribution in a molecule and strongly influence:
* Hydrogen bond strengths
* Long-range electrostatics
* Solvation free energies
* Binding free energies

Because electrostatics contribute significantly to **ΔG of binding**, an inconsistent or low-quality charge model can easily dominate the error 
budget — especially in **FEP/RBFE**, where small energy differences are amplified.

### Common charge models for small molecules

Below is a brief taxonomy of charge models you will encounter in practice.

#### 1. Gasteiger charges

Gasteiger charges are empirical charges computed by electronegativity equalization.

* Speed: Very fast

* Accuracy: Low to moderate

* Availability: Native in RDKit and many docking tools

* Typical use: Docking, virtual screening

In RDKit:
```
AllChem.ComputeGasteigerCharges(mol)
```

Gasteiger charges are widely supported by docking engines such as AutoDock4/Vina and are generally sufficient when:

* You only care about relative docking scores

* You are screening large libraries

* You want maximum throughput

They are **not appropriate** for production MD or FEP.

#### 2. MMFF94 partial charges

MMFF94 assigns partial charges as part of its molecular mechanics parameterization.

* **Speed**: Fast

* **Accuracy**: Moderate

* **Availability**: RDKit (via MMFFMolProperties)

* **Typical use**: Geometry optimization, rough energetics

These charges are force-field internal and were never intended to be transferable between different MD engines or force fields. 
While extractable, they are rarely used for serious binding free-energy work.

#### 3. UFF charges (or lack thereof)

UFF is primarily a geometry force field. In practice:

* There is no standard, physically meaningful UFF charge set

* RDKit does not expose a reliable per-atom UFF charge array

UFF is useful for initial geometry minimization, not electrostatics.

#### 4. AM1-BCC charges

**AM1-BCC** is the workhorse of modern ligand MD.

* **Method**: Semiempirical AM1 + bond charge corrections

* **Accuracy**: Close to RESP for many drug-like molecules

* **Cost*: Low

* **Typical use**: MD, RBFE/FEP

AM1-BCC charges are:

* Compatible with GAFF / GAFF2

* Fast enough for large ligand series

* Widely used in industry FEP pipelines
They are typically generated using AmberTools antechamber:
```bash
antechamber -i ligand.sdf -fi sdf \
            -o ligand.mol2 -fo mol2 \
            -c bcc -nc 0
```
