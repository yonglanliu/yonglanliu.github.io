
---
layout: post
title: "Absolute Hydration Free Energy with OpenMM"
---

## Motivation
Hydration free energy (ΔG_hyd) is a standard benchmark for validating
force fields and alchemical protocols before protein–ligand ABFE.

## Method
- OpenMM 8.x
- Alchemical decoupling in solvent
- Lambda schedule optimized for stability
- Multiple seeds for uncertainty estimation

## Design Rationale
This module is designed as a **reproducible pipeline entry point**:
- Configurable parameters
- Logged metadata
- Clear separation of system setup and simulation execution

## Code
GitHub:  
https://github.com/yonglanliu/openmm-rbfe

## Next Steps
- Extend to protein–ligand ABFE
- Integrate AI-based error prediction
