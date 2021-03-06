---
title: 'rustbca: A High-Performance Binary-Collision-Approximation Code for Ion-Material Interactions'
tags:
  - Rust
  - plasma material interactions
  - binary collision approximation
  - ion solid interactions
  - ion material interactions
  - sputtering
  - reflection
  - implantation
authors:
  - name: Jon. T Drobny
    orcid: 0000-0002-9733-6058
    affiliation: 1
  - name: Davide Curreli
    affiliation: 1
affiliations:
  - name: Department of Nuclear, Plasma, and Radiological Engineering, University of Illinois at Urbana-Champaign
    index: 1
date: 24 October 2020
bibliography: paper.bib
---

# Summary

Ion-material interactions are of vital importance in industrial applications, the study and design of nuclear fusion devices, engineering survivable spacecraft components, and more. In particular, plasma-material interactions, including the phenomena of sputtering, reflection, and implantation are dominated by ion-material interactions. These phenomena are difficult to model analytically, and many such models rely on empirical or semi-empirical formulas, such as the Yamamura sputtering yield formula [@YAMAMURA1982], or the Thomas reflection coefficient [@Thomas1992]. However, such models are inherently limited, and of little applicability to complex geometry, multi-component surfaces, or for coupling to plasma or material dynamics codes. Since ion-material interactions span a range of energies from sub-eV to GeV and beyond, n-body approaches such as molecular dynamics are computationally infeasible for many applications. Instead, approximations to the full n-body problem are used. Most commonly, the Binary Collision Approximation (BCA), a set of simplifying assumptions to the full n-body ion-material problem are used. rustbca is a high-performance, general purpose, ion-material interactions BCA code, built for flexibility and ease of use. rustbca includes 2D, inhomogeneous, arbitrary composition and geometry, electronic stopping formulations for low energy (up to 25 keV/nucleon) and high energy (up to 1 GeV/nucleon), Kr-C, ZBL, Moliere, and Lenz-Jensen screened coulomb interatomic potentials, Lennard-Jones and Morse attractive-repulsive potentials, the unique capability of using multiple interatomic potentials in a single simulation, choice of Gaussian quadrature and the approximate MAGIC algorithm for determining the scattering angle, full trajectory tracking of ions and material atoms, including local nuclear and electronic energy losses, a human-readable configuration file, and full 6D output of all particles that leave the simulation (via sputtering or reflection).

# Binary Collision Approximation Codes

rustbca is an amorphous-material BCA code. The TRIM[@Biersack1980] family of codes, which includes Tridyn[@Möller1988], SDTrimSP[@Mutzke2019], F-TRIDYN[@Drobny2017], and SRIM[@Ziegler2010], are also amorphous-material BCA codes, and historically the most popular implementation of the BCA. Based on the number of citations recorded in Google Scholar, SRIM, likely due to its being free to download, available on Windows, and having a graphical user interface, is the most popular amorphous-material BCA code, followed by the original TRIM, then Tridyn, and finally SDTrimSP. Crystalline-material BCA codes have also been developed, such as MARLOWE[@Robinson1974] and some versions of ACAT[@Yamamura1996], but are not as widely used. The BCA is a small set of simplifying assumptions for the ion-material interaction problem; the assumptions used in the amorphous-material BCA can be summarized as follows:

* Particles in the code, ions and material atoms both, are "superparticles" that represent many real ions or atoms each.
* Energetic particles interact with stationary atoms in the material through elastic, binary collisions
* Collisions occur at mean-free-path lengths
* Particle trajectories are approximated by the classical asymptotic trajectories
* Electronic interactions are separated from the nuclear, elastic interactions
* Local electronic energy losses occur at each collision
* Nonlocal electronic energy losses occur along each segment of the asymptotic trajectories
* Material atoms are mobile and transfer momentum following collisions
* Particles are stopped when their energy drops below a threshold, cutoff energy, or when they leave the simulation as sputtered or reflected/transmitted particles
* Particles that leave a surface experience reflection by or refraction through a locally planar surface binding potential
* When simulating radiation damage, only material atoms given an energy larger than the threshold displacement energy will be considered removed from their original location

For detailed summaries of the history and theory of binary collision approximation codes, see the reviews by Robinson[@Robinson1994] and Eckstein[@Eckstein1991].

# Statement of Need

Ion material interactions have been historically modeled using analytical and semi-empirical formulas, such as Sigmund's sputtering theory[@Sigmund1987], the Bohdansky formula[@Bohdansky1980; @Bohdansky1984], the Yamamura formula[@YAMAMURA1982; @Yamamura1983; @Yamamura1984], and the Thomas et al. reflection coefficient[@Thomas1992]. However, for any physical situation beyond the regimes of validity of these formulas, or for complex geometry or composition, or inhomogeneous composition, straightforward empirical formulas cannot be reliably used. Many BCA codes have been developed to provide computationally efficient solutions to these problems. Many of these, including SRIM[@Ziegler2010], Tridyn[@Möller1988], F-TRIDYN[@Drobny2017], SDTrimSP[@Mutzke2019] and its derivatives, are based on the original TRIM code. However, each has limitations that prevent widespread adoption across a broad range of applications. In particular, SRIM, which is free-use but closed-source, suffers from relatively poor computational performance and significant anomalies in sputtered atom angular distributions and light ion sputtering yields[@Shulga2019; @Shulga2018; @Hofsass2014]. Tridyn and F-TRIDYN, which are not open source, are limited to low energy, screened-coulomb potentials, mono-angular ion beams, atomically flat and atomically rough surfaces respectively, and are single-threaded. SDTrimSP, although significantly more developed than the preceding codes, is built on the original TRIM source code and is not open-source.

As far as the authors are aware, there is no widely-accepted open-source BCA code. Additionally, those that are available are not well suited to a wide range of problems, including direct coupling of BCA codes to particle and subsurface dynamics codes, such as those performed using F-TRIDYN for ITER divertor simulations[@Lasa2020]. rustbca has been developed to fill that gap and expand upon the feature set included in currently available BCA codes. Features unique to rustbca include the ability to handle attractive-repulsive interatomic potentials, including  the ability to use multiple interatomic potentials in one simulation, handling high-energy incident ions and 2D geometry, large file input of incident particles to facilitate coupling to other codes via HDF5, a human-readable, TOML configuration file, modern programming techniques, robust error-handling, and multi-threading. Rustbca is being developed as both a standalone code and, in the future, as a library code that can be used to add BCA routines to other high-performance codes. Largely, the TRIM family of codes relies on the MAGIC algorithm, developed by Ziegler, to approximate the scattering integral with 5 fitting coefficients. rustbca includes not only an implementation of the MAGIC algorithm, but also Mendenhall-Weller, Gauss-Mehler, and Gauss-Legendre quadrature, the three of which are significantly more accurate than the MAGIC algorithm. We hope that giving users direct access to a user-friendly, flexible, high-performance, open-source BCA will encourage and enable heretofore unexplored research in ion-materials interactions.

![Figure showing sputtering yields of silicon from SRIM, rustbca, F-TRIDYN, Yamamura's formula for Q=0.33-0.99, and a smooth analytical fit to experimental data by Wittmaack, for an incident energy of 1 keV and for many different projectiles.](corrected_yields.png)

This figure shows the sputtering yields of silicon by 1 keV helium, beryllium, oxygen, neon, aluminum, silicon, argon, titanium, copper, krypton, xenon, ytterbium, tungsten, gold, lead and uranium ions. SRIM's unphysical Z1 dependence is clearly visible, as is the divergence of Yamamura's formula (for Q = 0.66, the reported value for silicon, and +/- 0.33) at high mass ratios (M1 >> M2) from the experimental data collected by Wittmaack[@Wittmaack2004]. Rustbca and F-TRIDYN both reproduce the correct Z1 dependence of the sputtering yield, and correctly model the magnitude of the yield for all projectiles. It should be noted that, for this simulation, F-TRIDYN uses corrected MAGIC coefficients, reported in [@Ziegler2010]. A soft grey line depicts the point of silicon on silicon. Reflection coefficients, although very low, are also shown, with F-TRIDYN and rustbca agreeing with the semi-empirical Thomas reflection coefficient formula.

# Examples

rustbca includes multiple example input files, under the examples/ folder on the directory, as well as discussion of each on the rustbca github wiki page. Two examples can be run with the standard version of the code and will be summarized here. The final example requires the Chebyshev-Proxy Root-finder optional feature to solve the scattering integral for attractive-repulsive potentials.

First, an example of 2 keV helium ions at normal incidence on a layered titanium dioxide, aluminum, and silicon target can be run with:

 `cargo run --release examples/layered_target.toml`

 ![Helium implantation depth distributions at 2 keV in a layered TiO2-Al-Si target.](layered_target.png)

 The depth distribution, compared to F-TRIDYN, clearly shows the effect of layer composition on the combined nuclear and electronic stopping of helium.

 Second, as an example of the capability of rustbca to handle 2D geometry, the trajectories of 1 keV hydrogen on a circular cross-section of boron-nitride, meant to illustrate a boron-nitride dust grain can be simulated.

 `cargo run --release examples/boron_nitride.toml`

 ![Trajectories of hydrogen and mobile boron and nitrogen resulting from 10 1 keV hydrogen ions impacting on a circular cross-section boron-nitride target.](H_B_N.png)

# References
