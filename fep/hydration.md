---
layout: page
title: Hydration Free Energy (ΔG_hyd)
---

## Why hydration first?
Before running protein–ligand ABFE, I validate the protocol on **absolute hydration free energy** because it is:
- interpretable (single molecule → solvent)
- cheap compared to protein simulations
- sensitive to force field / lambda schedule / sampling issues

Hydration is a **protocol calibration stage**, not a final deliverable.

---

## What is computed?
ΔG_hyd is computed via alchemical decoupling of a ligand from water to vacuum (or a non-interacting reference),
typically separating:
- electrostatics (charge decoupling)
- van der Waals / LJ interactions (softcore to avoid singularities)

The core outputs:
- ΔG_hyd estimate
- uncertainty (e.g., via MBAR/BAR + replicate statistics)
- QC diagnostics

---

## Typical run design (conceptual)
A stable protocol usually includes:
- a lambda schedule with **denser spacing near endpoints**
- multiple seeds / replicates for uncertainty realism
- checkpointing for robustness
- consistent equilibration and production blocks per window

**Primary goal:** stable overlap between adjacent lambda states.

---

## Diagnostics I care about
### 1) Stability per window
- integration stability (no NaNs / blowups)
- temperature control reasonable
- potential energy behavior not pathological

### 2) Overlap & estimator robustness
- neighboring windows have sufficient overlap
- free energy contributions not dominated by a single problematic state

### 3) Replicate consistency
- different seeds should agree within uncertainty
- if not: increase sampling or revisit protocol

### 4) Convergence vs time
- ΔG(t) plateaus
- uncertainty decreases with more data

---

## Common failure modes (and how I respond)
- **Endpoint issues (lambda ~ 0 or 1):**
  - adjust softcore settings
  - add more windows near endpoints
- **Poor overlap:**
  - re-space lambda schedule
  - increase sampling or add intermediate states
- **High replicate variance:**
  - longer production, stronger equilibration, or better initial coordinates
- **Chemistry problems (bad ligand input):**
  - validate protonation/tautomer state
  - ensure correct bonding + atom typing before simulation

---

## How this informs ABFE readiness
I consider the protocol “ABFE-ready” if hydration runs show:
- stable windows
- reasonable overlaps across lambda
- replicates agree (uncertainty is believable)
- convergence trends make sense

Hydration is the **calibration gate** before moving to complex protein systems.

---

## Next step
Once hydration is validated, I apply the same protocol principles to ABFE:
- add restraints (where needed)
- include complex and solvent legs
- enforce strict reproducibility and diagnostics

See: `/fep/abfe/` (planned)

