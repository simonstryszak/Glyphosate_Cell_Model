# Bioengineered Glyphosate-Degrading Cell Model

A kinetic model of an engineered cellular system designed to break down glyphosate
(the active ingredient in many herbicides) into inert end products. The model
simulates the full enzymatic degradation cascade, compares different genetic-control
strategies for regulating enzyme expression, and tests how the system holds up under
different patterns of glyphosate exposure and under finite cellular resources.

Built in Google Colab. The entire model lives in Glyphosate_Cell_Model.ipynb.

Overview

The system models a population of synthetic cells that take up glyphosate from a
surrounding pool and run it through a six-step degradation pathway, tracking
metabolite concentrations, enzyme/mRNA expression, and the depletion of cellular
energy and building blocks over time.

The degradation pathway modeled is:

Glyphosate (external)
   │  GltT transporter (uptake)
   ▼
Glyphosate (internal)
   │  GOX (glyphosate oxidase)
   ▼
AMPA
   │  Transaminase (ping-pong mechanism, consumes pyruvate)
   ▼
CHOPO3 (formylphosphonate)
   │  Phosphonatase
   ▼
HCHO (formaldehyde)
   │  Formaldehyde dehydrogenase
   ▼
Formate
   │  Formate dehydrogenase
   ▼
Inert product  →  bulk remediation yield

Everything is expressed as a system of coupled ODEs and integrated with stiff
solvers (BDF / LSODA) from SciPy. Enzyme abundance is itself dynamic: each
enzyme has its own mRNA transcription and protein translation, both gated by
shared pools of cellular energy (E) and building blocks (B), so expression
and metabolism compete for the same finite resources.

What the model includes

Three regulation strategies


Constitutive — all degradation enzymes expressed at a fixed level.
PhnF biosensor — a glyphosate/AMPA-responsive repressor (PhnF) that turns
enzyme expression on only when substrate is present, modeled with a
Hill-type activation function. (Two variants: AMPA-sensing and internal-glyphosate-sensing.)
Expression-tuned — enzyme expression weights re-balanced across the pathway
(e.g. boosting the slowest step, formaldehyde dehydrogenase, and trimming the
fastest, phosphonatase) while holding total expression budget constant. Includes
a kinetics-optimized and a transport-optimized variant.


Four glyphosate delivery regimes


Bolus — a single initial dose, no further input.
Constant — steady inflow until a shutoff time.
Periodic — repeating pulses (sinusoidal half-waves).
Shock — a single large transient spike.


Resource & lifetime analysis
Tracks energy and building-block depletion and computes the functional "lifetime"
of the system as the point where either resource crosses a death threshold —
i.e. whichever runs out first kills the system.

Monte Carlo sensitivity scans


Whole-system scan over cell number (N_cells) with 95% confidence bands on yield.
Transaminase kinetics scan (kcat, Km) on AMPA → CHOPO3 conversion.
Phosphonatase kinetics scan using PhnX literature values.


Key state variables

The full state vector has 23 entries. The most important:

SymbolMeaningG_extexternal glyphosate concentrationG_intinternal glyphosate concentrationAMPA, CHOPO3, HCHO, Formatepathway intermediatesInert / Ybulkinert product and bulk remediation yieldGltT, GOX, Transaminase, Phosphonatase, FormaldehydeDH, FormateDHenzyme levels (each with a paired mRNA species)E, Bcellular energy and building-block poolsPyrpyruvate (co-substrate for the transaminase step)

Requirements

numpy
scipy
matplotlib

Install with:

bashpip install numpy scipy matplotlib

Running it

Open Glyphosate_Cell_Model.ipynb in Jupyter or Colab and run the cells top to
bottom. The first cells define the ODE system, parameters, and regulation/input
functions, then integrate every regulation-strategy × delivery-regime combination
and produce the figures (remediation yield, external glyphosate clearance,
sensor activation, transporter dynamics, resource depletion, and the Monte Carlo
bands). Later cells run the standalone enzyme-kinetics parameter scans and the
internal-glyphosate-sensing variant.

Parameters

Enzyme kinetics (kcat, Km) are drawn from literature/BRENDA values where
available (phosphonatase uses PhnX values, DOI: 10.1021/acs.biochem.1c00092).
Transcription/translation rates, degradation rates, resource-depletion
coefficients, cell number, and pool volume are set in the params dictionary at
the top of each model cell and can be edited there.

Notes & limitations


This is a deterministic kinetic model, not a spatial or stochastic one; it
assumes well-mixed compartments.
Several rate constants (especially transcription/translation and
resource-depletion terms) are estimates and are the main targets of the
sensitivity analysis.
Results are computational predictions intended to compare design strategies,
not validated experimental measurements.


Author

Simon Stryszak
Created in CoLab
