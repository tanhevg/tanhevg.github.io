---
layout: page
title: Projects
description: >
  Complete work that has already been published or otherwise disseminated.
hide_description: true
sitemap: false
permalink: /projects/
---

Complete work that has already been published or otherwise disseminated.

## Projects

* [RP3Net](https://github.com/RP3Net/RP3Net/) - June 2025
  * An AI model for predicting recombinant protein production. Work done at EBI in collaboration with AstraZeneca. Published in [Bioinformatics](https://doi.org/10.1093/bioinformatics/btag003)

* [GpABC.jl](https://github.com/tanhevg/GpABC.jl) - Feb 2020
  * A Julia library for likelihood-free Bayesian parameter estimation and model selection. Uses Approximate Bayesian Computation (ABC) and features an emulation mode, where model simulations in ABC are replaced with emulation with Gaussian Processes (GP). Linear Noise Approximation (LNA) is provided for models that violate Gaussian priors. This work was done as a group project for MSc course in Bioinformatics at Imperial College London, and was published in [Bioinformatics journal](https://doi.org/10.1093/bioinformatics/btaa078)

![GpABC](/assets/img/projects/gpabc.png){:.lead width="800" height="600" loading="lazy"}

Parameter estimation for the "three genes" example with GpABC. Subplots on the diagonal and below show marginal and joint posterior distributions in the final ABC-SMC population (simulation in blue and emulation in red). Scatterplots above the diagonal show intermediate ABC-SMC populations with GP emulations; darker colour indicates decreasing threshold Black dashed lines indicate the true parameter values.
{:.figcaption style="text-align: left"}

* [Cool compiler](https://github.com/tanhevg/cool-compiler) - Jun 2016
  * `COOL` is a "classroom object oriented language" - a language developed by [Alex Aiken](https://theory.stanford.edu/~aiken/software/) for teaching his compiler course. I wanted to understand how a compiler works, so I built a complier for COOL, following the instructions from the course. It features a lexer, a type checker and a code generator that spews out MIPS assembly. The assembly can then be executed on a [spim emulator](https://sourceforge.net/projects/spimsimulator/). The compiler is written in C++ 
