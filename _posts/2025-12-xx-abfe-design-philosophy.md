---
layout: post
title: "From Single-Edge FEP to Multi-Edge Ligand Networks: Building a Modern Free Energy Pipeline"
date: 2025-12-18
tags: [free-energy, alchemical, FEP, RBFE, OpenMM, OpenFE, computational-chemistry]
---

> This post documents my own journey from writing a simple one-edge FEP script in OpenMM  
> to understanding how modern industrial workflows (OpenFE, FEP+) use **ligand networks**  
> to scale relative binding free energy (RBFE) calculation to dozens of ligands.

Over the past few months, I have been building my own OpenMM-based free energy pipeline.  
At first, everything was centered around **one transformation**:
                    Ligand A → Ligand B

That is the classic **single-edge** free energy perturbation (FEP).  
It works beautifully for hydration ΔG or small pairwise relative free energies.

But once I began studying real RBFE workflows—Schrödinger FEP+, OpenFE, Perses—it became clear:

> Modern RBFE is not about a single A→B transformation.  
> It is about building a **ligand mutation network (graph)**  
> and using many small, stable edges to compute ΔG for an entire ligand series.

This blog post introduces that transition step by step:

1. What single-edge FEP actually computes  
2. Why single-edge is not enough for drug discovery  
3. What a ligand network is (with diagrams)  
4. How a single-edge pipeline evolves into a network-based RBFE engine  
5. How this relates to OpenFE and industrial workflows

---

# 1. Single-Edge FEP: The Minimal Unit

Let’s start with the simplest alchemical calculation:  
**transforming ligand A into ligand B**.
A → B

This requires two legs:

- **complex leg**: `protein + A → protein + B`
- **solvent leg**: `A in water → B in water`

Then:

\[
\Delta\Delta G = \Delta G_{\text{complex}} - \Delta G_{\text{solvent}}
\]

**ASCII diagram:**

Complex:
[ Protein + A ] -- ΔG_complex --> [ Protein + B ]

Solvent:
[ A in water ] -- ΔG_solvent --> [ B in water ]

Final:
ΔΔG = ΔG_complex - ΔG_solvent

My personal OpenMM pipeline already handles this:

- system building  
- minimization + equilibration  
- lambda schedule  
- MD propagation  
- BAR/MBAR analysis  

This works well—but it is still **one edge**.

---

# 2. Why Single-Edge FEP Is Not Enough

In real drug discovery projects, you rarely have “A and B”.  
You have something like:

\[
L_1, L_2, L_3, \ldots, L_{20}
\]

If you try to evaluate all ligands using single-edge FEP:

- you must decide manually which pairs to transform  
- some mutations are too large to converge  
- number of possible pairs scales as \(O(N^2)\)  
- you lose the advantage of comparing “similar ligands”

In practice, drug discovery teams want:

- fast evaluation of a whole ligand series  
- stability  
- cycle closure checking (self-consistency)  
- ability to choose the best growing direction for hit-to-lead

This is where single-edge FEP collapses.

The solution is **ligand networks**.

---

# 3. Ligand Networks: Nodes, Edges, Graphs

A ligand network is simply a graph:

- **nodes = ligands**  
- **edges = alchemical transformations**  

Here is a minimal example with four ligands:

```mermaid
graph LR
    L1 --- L2
    L2 --- L3
    L3 --- L4
    L1 --- L4
What this means:

Each line (edge) is one FEP calculation (A→B).

The graph connects similar ligands first (easy transformations).

Once all edges are computed, you can reconstruct
relative binding free energies for all ligands simultaneously.

Why networks work:

Avoid large transformations

Ligands mutate only to similar ligands → faster convergence

Cycle closure

If L1→L2→L3→L4→L1 ≈ 0

The network is self-consistent

Global fit

You can choose any ligand as reference:

Δ
𝐺
bind
(
𝐿
𝑖
)
−
Δ
𝐺
bind
(
𝐿
ref
)
ΔG
bind
	​

(L
i
	​

)−ΔG
bind
	​

(L
ref
	​

)

This is exactly what industrial platforms do:

Schrödinger FEP+

OpenFE

Perses (OpenMM-based)

NAMD/AMBER RBFE workflows



