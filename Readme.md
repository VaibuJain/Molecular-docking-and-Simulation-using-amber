# Protein–Ligand Molecular Docking and Molecular Dynamics Simulation

This repository contains a computational workflow for **protein–ligand molecular docking followed by molecular dynamics (MD) simulation**.

The workflow combines **AutoDock4** for molecular docking, **ACPYPE** for ligand topology generation, **PDBFixer** for protein structure preparation, and **GROMACS** for molecular dynamics simulations and trajectory analysis.

The workflow includes protein and ligand preparation, docking, ligand topology generation, system assembly, solvation, ion addition, energy minimization, NVT and NPT equilibration, 100-ns production MD simulation, and post-simulation structural analysis.

---

## Workflow Overview

The complete workflow follows these major steps:

```text
Protein Preparation
        │
        ▼
Ligand Preparation
        │
        ▼
Molecular Docking
   (AutoDock4)
        │
        ▼
Docking Pose Selection
        │
        ▼
Protein–Ligand Complex
        │
        ▼
Ligand Topology
      (ACPYPE)
        │
        ▼
Protein Preparation
    (PDBFixer)
        │
        ▼
System Assembly
        │
        ▼
Simulation Box
        │
        ▼
Solvation
        │
        ▼
Ion Addition
        │
        ▼
Energy Minimization
        │
        ▼
NVT Equilibration
        │
        ▼
NPT Equilibration
        │
        ▼
100-ns Production MD
        │
        ▼
Trajectory Processing
        │
        ├── RMSD
        ├── RMSF
        ├── Radius of Gyration
        ├── Hydrogen-Bond Analysis
        └── Energy Analysis
```

---

## Repository Contents

```text
protein-ligand-md-simulation/
│
├── README.md
│
├── protein_ligand_MD_workflow.ipynb
│
└── mdp/
    ├── ions.mdp
    ├── minim.mdp
    ├── nvt.mdp
    ├── npt.mdp
    └── 100ns.mdp
```

### Files

| File                               | Description                                  |
| ---------------------------------- | -------------------------------------------- |
| `protein_ligand_MD_workflow.ipynb` | Complete step-by-step computational workflow |
| `mdp/ions.mdp`                     | Parameters used for ion preparation          |
| `mdp/minim.mdp`                    | Energy minimization parameters               |
| `mdp/nvt.mdp`                      | NVT equilibration parameters                 |
| `mdp/npt.mdp`                      | NPT equilibration parameters                 |
| `mdp/100ns.mdp`                    | Production MD simulation parameters          |
| `README.md`                        | Documentation and workflow description       |

---

# Software Requirements

The workflow uses the following computational tools:

* **AutoDock4**
* **AutoGrid4**
* **MGLTools / AutoDockTools**
* **Open Babel**
* **ACPYPE**
* **PDBFixer**
* **GROMACS**
* **Python**
* **Matplotlib** for plotting and visualization
* **PyMOL** for structural inspection and visualization

The exact versions of the software may affect the results. It is recommended to record the versions used for a particular study.

---

# Input Requirements

The protein and ligand structures used for the original analysis are **not included in this repository**.

Users should provide their own protein and ligand structures.

## Protein

A protein structure in PDB format is required.

For example:

```text
protein.pdb
```

The protein structure can be obtained from an experimental structure database such as the Protein Data Bank or from a predicted structure, depending on the research application.

Before docking, the protein should be inspected for:

* unwanted ligands
* water molecules
* alternate conformations
* missing residues/atoms
* non-standard residues
* structural inconsistencies

---

## Ligand

A ligand structure is required for docking and topology generation.

The workflow can use ligand structures prepared from sources such as PubChem or other chemical databases.

Example:

```text
ligand.pdb
```

The ligand is subsequently converted/prepared for docking and molecular dynamics.

---

# 1. Protein and Ligand Preparation

The first stage involves preparing the protein and ligand structures for molecular docking.

The protein should be checked for unwanted molecules or additional ligands.

The ligand structure should be converted into an appropriate format and prepared with the required hydrogens and atomic charges.

---

# 2. Molecular Docking

Molecular docking is performed using:

* AutoDock4
* AutoGrid4
* AutoDockTools

The docking workflow includes:

1. Preparation of receptor and ligand files
2. Generation of the grid parameter file (`.gpf`)
3. Generation of the docking parameter file (`.dpf`)
4. AutoGrid calculation
5. AutoDock calculation
6. Examination of docking poses

The docking results are inspected and the preferred binding pose is selected based on docking score/energy and structural inspection.

The selected protein–ligand complex is subsequently used for molecular dynamics simulation.

---

# 3. Docking Pose Inspection

The selected docking pose can be visualized using **PyMOL**.

The docking output is converted into an appropriate structural format before further processing.

The protein and ligand are separated from the selected complex for subsequent topology preparation.

---

# 4. Ligand Topology Generation Using ACPYPE

The ligand topology required for GROMACS is generated using **ACPYPE**.

The ligand is first converted into an appropriate molecular format, followed by topology generation.

Example workflow:

```bash
obabel ligand_only.pdb -O ligand_only.mol2 --gen3d
```

Then ACPYPE is used to generate the GROMACS-compatible ligand topology:

```bash
acpype -i ligand_only.mol2 -c user -n 0
```

The generated files include ligand topology and coordinate files required for GROMACS simulations.

---

# 5. Protein Preparation

The protein structure is processed using **PDBFixer** to address structural issues such as missing atoms, residues, and non-standard residues where appropriate.

The processed protein is subsequently converted into a GROMACS-compatible topology using:

```bash
gmx pdb2gmx
```

The workflow uses the:

```text
AMBER99SB-ILDN
```

force field with:

```text
TIP3P
```

water.

---

# 6. Protein–Ligand System Assembly

The processed protein and ligand coordinate files are combined to construct the protein–ligand simulation system.

The ligand topology is included in the main GROMACS topology file:

```text
#include "ligand_only.acpype/ligand_only_GMX.itp"
```

The topology must also contain the appropriate ligand molecule definition.

> **Important:** The protein and ligand coordinate files are not provided in this repository. Users must modify the filenames and topology entries according to their own system.

---

# 7. Simulation Box Definition

A simulation box is generated around the protein–ligand complex using GROMACS.

The workflow uses a cubic simulation box with approximately 1.0 nm distance between the complex and the box boundary.

Example:

```bash
gmx editconf -f complex.gro -o boxed.gro -c -d 1.0 -bt cubic
```

---

# 8. Solvation

The simulation box is solvated using the GROMACS solvent configuration.

Example:

```bash
gmx solvate -cp boxed.gro -cs spc216.gro -o solv.gro -p topol.top
```

The solvent molecules are added and the topology is updated accordingly.

---

# 9. Ion Addition

The solvated system is prepared for ion addition using the `ions.mdp` parameter file.

The system is then converted into a portable binary run input file (`.tpr`):

```bash
gmx grompp -f ions.mdp -c solv.gro -p topol.top -o ions.tpr
```

Counterions are subsequently added to neutralize the system:

```bash
gmx genion -s ions.tpr -o solv_ions.gro -p topol.top \
-pname NA -nname CL -neutral
```

---

# 10. Energy Minimization

Energy minimization is performed to remove unfavorable atomic contacts and steric clashes introduced during system preparation.

The workflow uses the `minim.mdp` parameter file.

Example:

```bash
gmx grompp \
-f minim.mdp \
-c solv_ions.gro \
-p topol.top \
-o em.tpr
```

The minimization is then performed using:

```bash
gmx mdrun -v -deffnm em
```

The minimized system is subsequently inspected using potential-energy analysis.

---

# 11. NVT Equilibration

The system is equilibrated under the **constant number of particles, volume, and temperature (NVT)** ensemble.

The `nvt.mdp` file contains the parameters used for NVT equilibration.

Example:

```bash
gmx grompp \
-f nvt.mdp \
-c em.gro \
-r em.gro \
-p topol.top \
-o nvt.tpr
```

The simulation is then performed using:

```bash
gmx mdrun -deffnm nvt
```

Temperature and other relevant energy terms are monitored to evaluate equilibration.

---

# 12. NPT Equilibration

Following NVT equilibration, the system is equilibrated under the **constant number of particles, pressure, and temperature (NPT)** ensemble.

Example:

```bash
gmx grompp \
-f npt.mdp \
-c nvt.gro \
-r nvt.gro \
-t nvt.cpt \
-p topol.top \
-o npt.tpr
```

The simulation is performed using:

```bash
gmx mdrun -deffnm npt
```

During NPT equilibration, properties such as:

* temperature
* pressure
* volume
* density
* potential energy

are monitored.

---

# 13. Production Molecular Dynamics

After equilibration, the system is subjected to **100 ns of production molecular dynamics simulation**.

The production simulation is prepared using:

```text
100ns.mdp
```

Example:

```bash
gmx grompp \
-f 100ns.mdp \
-c npt.gro \
-t npt.cpt \
-p topol.top \
-o md_100ns.tpr
```

Production MD is then performed using:

```bash
gmx mdrun -deffnm md_100ns
```

The resulting trajectory can be processed for downstream structural and interaction analyses.

---

# 14. Trajectory Processing

The production trajectory is converted to a compact trajectory format for analysis.

Example:

```bash
gmx trjconv \
-s md_100ns.tpr \
-f md_100ns.trr \
-o traj.xtc
```

The resulting trajectory can be used for structural stability and flexibility analyses.

---

# 15. Molecular Dynamics Analyses

The workflow includes several commonly used MD analyses.

## RMSD

Root Mean Square Deviation (RMSD) is calculated to assess structural stability throughout the simulation.

```bash
gmx rms \
-s md_100ns.tpr \
-f traj.xtc \
-o rmsd.xvg \
-tu ns
```

RMSD profiles can be used to evaluate the overall conformational stability of the protein–ligand system.

---

## RMSF

Root Mean Square Fluctuation (RMSF) is calculated to investigate residue-level flexibility.

```bash
gmx rmsf \
-s md_100ns.tpr \
-f traj.xtc \
-o rmsf.xvg
```

RMSF can identify flexible and relatively rigid regions of the protein.

---

## Radius of Gyration

The radius of gyration (`Rg`) is calculated to evaluate changes in the overall compactness of the protein.

```bash
gmx gyrate \
-s md_100ns.tpr \
-f traj.xtc \
-o gyrate.xvg
```

---

## Protein–Ligand Hydrogen Bonds

Hydrogen-bond interactions between the protein and ligand are analyzed using GROMACS hydrogen-bond analysis.

The resulting hydrogen-bond profile can be used to evaluate the persistence of polar interactions during the simulation.

---

## Energy Analysis

Energy terms can be extracted from GROMACS energy files using:

```bash
gmx energy
```

The workflow includes analysis of properties such as:

* potential energy
* kinetic energy
* total energy
* temperature
* pressure
* volume
* density

These measurements can be used to evaluate system equilibration and simulation behavior.

---

# MDP Parameter Files

The repository contains the GROMACS parameter files used at different stages of the workflow:

```text
ions.mdp
minim.mdp
nvt.mdp
npt.mdp
100ns.mdp
```

These files define the simulation parameters for ion preparation, energy minimization, equilibration, and production MD.

Users should review and adapt these parameters according to:

* protein size
* ligand properties
* force field
* simulation objectives
* system composition
* GROMACS version
* computational resources

---

# Reproducibility

The purpose of this repository is to provide a **reproducible computational workflow** for protein–ligand molecular docking and molecular dynamics simulation.

Because the original protein and ligand structures are not distributed with this repository, users should provide their own input structures and update the filenames/paths in the notebook.

For reproducibility, users should record:

* GROMACS version
* AutoDock version
* ACPYPE version
* Open Babel version
* PDBFixer version
* Python version
* operating system
* force field
* ligand charge and parameterization method
* simulation parameters

---

# Important Notes

### 1. Input structures are not included

The original protein and ligand structures are intentionally not included in this repository.

Users should supply their own structures.

### 2. File paths

The notebook should be adapted to the user's local directory structure.

Paths from the original computational environment should **not** be copied directly.

### 3. Computational requirements

Molecular dynamics simulations can require substantial computational resources, particularly for long production simulations.

Simulation performance depends on:

* CPU
* GPU
* available memory
* number of MPI/OpenMP threads
* GROMACS configuration

### 4. Parameter modification

The supplied `.mdp` files represent the parameters used for this workflow. They should be reviewed before applying the workflow to a different protein–ligand system.

### 5. Validation

Docking results and MD trajectories should be evaluated using appropriate structural, energetic, and interaction-based criteria rather than relying on a single metric.

---

# Citation and References

This workflow was developed using established molecular docking and molecular dynamics tools and protocols.

Users should cite the relevant software and methodological references when using this workflow in research.

Key software includes:

* AutoDock4
* AutoGrid4
* MGLTools/AutoDockTools
* ACPYPE
* Open Babel
* PDBFixer
* GROMACS

The original workflow/tutorial that informed parts of the procedure is:

**Angelo Raymond Rossi — CHARMM/GROMACS small organic molecule workshop**

https://angeloraymondrossi.github.io/workshop/charmm-gromacs-small-organic-molecules-new.html

Please also cite the original publications associated with the software used in your analysis.

---

# License

This repository can be distributed under an appropriate open-source license.

For example, if you want to allow reuse and modification with attribution, you may use the **MIT License**.

---

# Author

**Vaibhavi Jain**

This repository is intended to document and share a protein–ligand molecular docking and molecular dynamics simulation workflow for reproducible computational research.

