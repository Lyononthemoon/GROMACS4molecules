# GROMACS4molecules
## 1. Download the molecular structure from Pubchem.
*Tip: You can also build such structures by using Gaussian or Avogadro etc. directly or copy the smiles to the Avogadro (Under the "Build" menu, hold your cursor over "Insert", and select "SMILES...".)*

## 2. Optimize the molecule.sdf file by using ORCA based on B97-3c (enough).
*Tip: Use the Multiwnf to generation the ORCA input file (oi)*

Edit maxcore 5000 pal nprocs 16 end

## 3. Convert the .out file to PDB file
Usage Avogadro

Remember revise the "residues" UNL to specific name, like "baicalin" to "BCL"
## 4. Generate the .top file
Usage [GMXTOP](https://jerkwin.github.io/prog/gmxtop.html)

*Tip: only for OPLSAA forcefield*

Usage acpype.py

`./acpype.py -i molecule.mol2`

Usage [sobtop](http://sobereva.com/soft/Sobtop/)  (Recommended)

## 5. Create and Edit the .itp file
Edit the MOL to the specific name, like "baicalin" to "BCL"
## 6. Build the initial configuration

Usage Packmol (Recommended)

`Packmol < input.inp`

or usage the `gmx insert-molecules` command line

## 7. Run GROMACS
### 7.1 solvate
`gmx solvate -cp "box".gro -cs spc216.gro -o solv.gro -p topol.top`
### 7.2 genions
`gmx grompp -f ions.mdp -c solv.gro -p topol.top -o ions.tpr`

`echo 13 | gmx genion -s ions.tpr -o solv_ions.gro -p topol.top -pname NA -nname CL -neutral`
### 7.3 minimal energy
`gmx grompp -f em.mdp -c solv_ions.gro -p topol.top -o em.tpr`

`gmx mdrun -deffnm em -v`
### 7.4 nvt
`gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr`

`gmx mdrun -deffnm nvt -v`
### 7.5 npt
`gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -p topol.top -o npt.tpr`

`gmx mdrun -deffnm npt -v`

### 7.6 mdrun
`gmx grompp -f md.mdp -c npt.gro -t npt.cpt -p topol.top -o md_0_1.tpr`

`gmx mdrun -deffnm md_0_1 -v -ntmpi 6 -ntomp 8`

### 7.7 Trjconv
`gmx make_ndx -f npt.gro -o index.ndx`

`gmx trjconv -s md_0_1.tpr -f md_0_1.xtc -o md_0_1_noPBC.xtc -pbc cluster -center -n index.ndx`

`gmx trjconv -s md_0_1.tpr -f md_0_1_noPBC.xtc -o end.pdb -dump 200000 -n index.ndx`
### 7.8 analysis
`gmx rms -s md_0_1.tpr -f md_0_1_noPBC.xtc -n index.ndx -o rmsd.xvg -b 0 -e 50 -dt 0.1 -tu ns`

`gmx gyrate -s md_0_1.tpr -f md_0_1_noPBC.xtc -n index.ndx -o gyrate.xvg`

### DynamiSpectra
*is an advanced Python package designed for analyzing molecular dynamics simulation data.

#### Online document

[Contents — DynamiSpectra 1.1.0 documentation](https://conradoou.github.io/DynamiSpectra/index.html)

#### Online

[DynamiSpectra - Home](https://dynamispectra.onrender.com/)
`gmx rdf -s md_0_1.tpr -f md_0_1_noPBC.xtc -n index.ndx -o rdf.xvg -b 0 -e 200 -dt 0.1`

`gmx sasa -f md_0_1_noPBC.xtc -s md_0_1.tpr -n index.ndx -tu ns -odg -surface -o area.xvg -b 0 -e 50 -dt 0.1`

`gmx hbond -s md_0_1.tpr -f md_0_1_noPBC.xtc -n index.ndx -num hbnum.xvg -tu ns`

