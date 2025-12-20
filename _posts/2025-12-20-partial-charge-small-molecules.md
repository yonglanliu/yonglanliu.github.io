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

* **Cost**: Low

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
Using the [OpenFF](https://github.com/openforcefield/openff-toolkit) workflow to assign AM1-BCC charge. **OpenFF**
```bash
import openmm.app as app
from openmmforcefields.generators import GAFFTemplateGenerator
forcefield = app.ForceField("amber14/tip3p.xml")
gaff = GAFFTemplateGenerator(molecules=[offmol], forcefield="gaff-2.11")
forcefield.registerTemplateGenerator(gaff.generator)
```
GAFFTemplateGenerator (from openmmforcefields) uses AmberTools (antechamber/parmchk2) to:
* assign GAFF atom types
* compute AM1-BCC partial charges (default behavior for this generator)
* generate parameters/templates injected into the OpenMM ForceField

Then forcefield.createSystem(...) builds the OpenMM System using those parameters, including the ligand charges.

**How to confirm charges are present (quick check)**
After system = forcefield.createSystem(...), you can inspect the NonbondedForce:
```bash
from openmm import NonbondedForce

nb = next(f for f in system.getForces() if isinstance(f, NonbondedForce))
charges = [nb.getParticleParameters(i)[0].value_in_unit(unit.elementary_charge)
           for i in range(system.getNumParticles())]

print("Total charge:", sum(charges))
print("First 10 charges:", charges[:10])
```

#### 5. RESP charges

**RESP (Restrained Electrostatic Potential)** charges are derived by fitting atomic charges to reproduce a **quantum-mechanical electrostatic potential**.
* **Accuracy**: Very high
* **Cost**: Expensive (QM calculation required)
* **Typical use**: High-accuracy MD, challenging chemotypes
RESP is particularly useful when:
* The molecule has unusual charge delocalization
* Metal coordination is involved
* Publication-grade accuracy is required
The traditional RESP workflow involves:
1. QM ESP calculation (e.g., HF/6-31G*)
2. Two-stage restrained fitting using AmberTools
RESP is often overkill for routine FEP but invaluable for edge cases.

#### Docking vs. FEP: what should you use?
The most common mistake is using the same charge model for docking and free-energy calculations. These tasks have very different requirements.
*Docking / virtual screening*

Recommended: Gasteiger
* Fast
* Widely supported
* Consistent with scoring functions
Docking scores are approximate by nature; the marginal gain from higher-level charges rarely justifies the cost.

#### MD / RBFE / FEP
Recommended: AM1-BCC
Alternative: RESP (when needed)
For free-energy calculations:
* Electrostatics must be physically meaningful
* Charges must be consistent across alchemical transformations
* Mixing charge models across ligands will destroy convergence
* Using Gasteiger charges in FEP is almost guaranteed to produce noisy, unreliable ΔG values.

### Practical recommendations

Rule of thumb:

|Task	| Charge model|
|-----|-------------|
|Docking / VS	|Gasteiger|
|Classical MD	|AM1-BCC |
|RBFE / FEP	| AM1-BCC|
|High-accuracy studies	|RESP |

This full code for partial charge assignment can be found in my [this repository]()
