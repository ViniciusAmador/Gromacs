![License](https://img.shields.io/badge/license-MIT-green)
![GROMACS](https://img.shields.io/badge/GROMACS-2023-blue)
![MD Analysis](https://img.shields.io/badge/MD-Analysis-brightgreen)

# Generic Molecular Dynamics Protocol For Globular Proteins Using GROMACS
## Professor, Dr. Vinícius Costa Amador 
## Objective
This document provides a comprehensive step-by-step protocol for performing molecular dynamics simulations using GROMACS. Each section includes command explanations, file outputs, and updated directory structures after each major step.

```bash
project/
├── input/         # PDB and parameter files
└── scripts/       # Analysis scripts
```

# 1. System Preparation
Prepare the initial structure with hydrogens and a defined force field for simulation.

## 1.1. Adding Hydrogens and Choosing a Force Field
```bash
gmx pdb2gmx -f input.pdb -o processed.gro -water tip3p
```
Generated files: processed.gro, topol.top, posre.itp

Updated Directory Structure
```bash
project/
├── input/
├── processed/     # processed.gro, topol.top, posre.itp
└── scripts/
```
# 2. Defining the Simulation Box
Center the molecule and define an appropriate simulation box size and type.
```bash
gmx editconf -f processed.gro -o boxed.gro -c -d 1.0 -bt dodecahedron
```
Generated file: boxed.gro

# 3. Solvating the System
Fill the simulation box with solvent molecules.
```bash
gmx solvate -cp boxed.gro -cs spc216.gro -o solvated.gro -p topol.top
```
Generated file: solvated.gro

Updated Directory Structure
```bash
project/
├── input/
├── processed/     # boxed.gro, solvated.gro, topol.top, posre.itp
└── scripts/
```
# 4. Adding Ions
Neutralize the solvated system with counterions.
```bash
gmx grompp -f ions.mdp -c solvated.gro -p topol.top -o ions.tpr
gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -neutral
```
Generated files: ions.tpr, solv_ions.gro

# 5. Energy Minimization
Relax the system by removing steric clashes and high-energy contacts.
```bash
gmx grompp -f minim.mdp -c solv_ions.gro -p topol.top -o em.tpr
gmx mdrun -deffnm em
```
Generated files: em.gro, em.edr, em.log, em.trr

Updated Directory Structure
```bash
project/
├── input/
├── processed/     # em.gro, em.edr, em.log, em.trr, boxed.gro, solvated.gro, topol.top
└── scripts/
```
# 6. Equilibration
Stabilize temperature and pressure before production.
```bash
gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr
gmx mdrun -deffnm nvt

gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -p topol.top -o npt.tpr
gmx mdrun -deffnm npt
```
# 7. Production Molecular Dynamics
Perform the full molecular dynamics simulation.
```bash
gmx grompp -f md.mdp -c npt.gro -p topol.top -o md.tpr
gmx mdrun -deffnm md
```
Generated files: md.xtc, md.edr, md.log, md.gro

Updated Directory Structure
```bash
project/
├── input/
├── processed/     # em.gro, nvt.gro, npt.gro, md.gro, and other intermediate files
├── md/            # md.xtc, md.edr, md.log
└── scripts/
```
# 8. Basic Analyses
Evaluate structural stability and basic properties post-simulation.
```bash
gmx rms -s md.tpr -f md.xtc -o rmsd.xvg
gmx gyrate -s md.tpr -f md.xtc -o gyrate.xvg
gmx hbond -s md.tpr -f md.xtc -num hbnum.xvg
gmx rmsf -s md.tpr -f md.xtc -o rmsf.xvg
gmx energy -f md.edr -o energy.xvg
gmx trjconv -f md_0_1noPBC.trr -s md_0_1.tpr -o filme.pdb -ur compact -center -pbc mol
gmx hbond -f md_0_1noPBC.trr -s md_0_1.tpr -hbm hmap.xpm -hbn hbonds.ndx -b 1 -e 10000
```
Generated files: rmsd.xvg, gyrate.xvg, hbnum.xvg, rmsf.xvg, energy.xvg, filme.pdb, hmap.xpm, hbonds.ndx

Updated Directory Structure
```bash
project/
├── input/
├── processed/
├── md/            # md.xtc, md.edr, md.log
├── analysis/      # rmsd.xvg, gyrate.xvg, hbnum.xvg, rmsf.xvg, energy.xvg
└── visualization/ # filme.pdb, hmap.xpm
```
# 9. Advanced Analyses

Extract detailed insights into structural fluctuations, solvent accessibility, secondary structure, and residue-residue interactions.
```bash
gmx rmsf -f md_0_1noPBC.trr -s md_0_1.tpr -oq bfactor.pdb -ox bfactor_average.pdb -res
gmx do_dssp -f md_0_1noPBC.trr -s md_0_1.tpr -tu ns
gmx xpm2ps -f ss.xpm -o ss.eps -by 5
ps2pdf ss.eps ss.pdf
gmx sasa -s md_0_1.tpr -f md_0_1.trr -o SASA.xvg
gmx mdmat -f D3_md_0_1noPBC.trr -s D3_md_0_1.tpr -no D8.xvg
gmx xpm2ps -f dm.xpm -o D8.eps
```
Generated files: bfactor.pdb, bfactor_average.pdb, ss.pdf, SASA.xvg, D8.eps

Final Directory Structure
```bash
project/
├── input/
├── processed/
├── md/
├── analysis/      # All analysis results including SASA.xvg, bfactor.pdb, etc.
└── visualization/ # filme.pdb, secondary structure plots, contact maps
```
Notes

Validate minimization and equilibration steps.
Ensure required software (GROMACS, DSSP, Ghostscript) is installed.
Recommended visualization tools: PyMOL, VMD, Grace.


