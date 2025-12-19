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

The workflow covers the complete RBFE pipeline, from ligand preparation to result analysis.

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

## Workflow Outline

1. Ligand preparation  
2. Protein–ligand complex preparation  
3. Atom mapping and ligand network construction  
4. System setup  
5. Alchemical transformations  
6. RBFE calculations  
7. Lambda scheduling  
8. Result analysis  

### 1. Ligand Preparation

Ligand preparation ensures chemical consistency across all transformations.

Key steps include:

* Protonation state assignment
* Geometry optimization
* Partial charge assignment
* Format conversion to OpenFE-compatible objects
