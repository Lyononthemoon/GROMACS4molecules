# GROMACS4molecules
## 1. Download the molecular structure from Pubchem.

*Tip: You can also build such structures by using Gaussian or Avogadro etc. directly or copy the smiles to the Avogadro (Under the "Build" menu, hold your cursor over "Insert", and select "SMILES...".)*

## 2. Optimize the molecule.sdf file by using ORCA based on B97-3c (enough).

*Tip: Use the Multiwnf to generation the ORCA input file (oi)*

Edit maxcore 5000 pal nprocs 16 end

## 3. Convert the .out file to PDB file by using Avogadro.

## 4. Generate the .top file by using [GMXTOP](https://jerkwin.github.io/prog/gmxtop.html)
