# GROMACS4molecules
1. Download the molecular structure from Pubchem.

Tip: You can also build such structure by using Gaussian or Avogadro etc. directly or cope the smiles to the Avogadro (Under the "Build" menu, hold your cursor over "Insert", and select "SMILES...".)

3. Optimize the molecule.sdf file by using ORCA on the basis of B97-3c (enough).

Tip: Use the Multiwnf to generation the ORCA input file (oi)

Edit maxcore 5000 pal nprocs 16 end
