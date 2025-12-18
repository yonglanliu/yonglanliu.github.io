---
layout: page
title: OpenMM-based Free Energy Pipeline (openmm-rbfe)
---

## TL;DR
This project is a **reproducible, pipeline-first OpenMM workflow** for alchemical free energy calculations in drug discovery.
It is designed to support both:

- **Absolute hydration free energy (ΔG_hyd)** as a force-field & protocol benchmark
- **Protein–ligand ABFE (ΔG_bind)** as a physics-based decision tool for lead optimization

The guiding philosophy is: **benchmark → validate → scale → extend**.

---

## Why this pipeline exists
Free energy methods are powerful, but in practice they fail when workflows are:
- hard to reproduce (parameters/seeds not tracked)
- hard to debug (no standard diagnostics)
- hard to scale (manual prep, inconsistent outputs)
- hard to extend (ABFE built without calibration)

This repo aims to solve that by providing a **standardized entry point** for free energy runs, with
clear separation between:
1) system preparation,

