IsoMif - Readme
Matthieu Chartier, march 2015, Matthieu.Chartier@uSherbrooke.ca
Developped at the Najmanovich Research Group bcb.med.usherbrooke.ca under supervision of Rafael Najmanovich Rafael.Najmanovich@uSherbrooke.ca
University of Sherbrooke
Faculty of Medicine and Health Sciences
Québec, Canada

To run IsoMif locally on mac or linux, you need 3 programs (GetCleft, Mif and IsoMif). All included in the download. Use the binary files for your architecture. You can also compile the Mif and IsoMIF program on your machine.

################

1. Find Cavities

################


To download the PDB for the example you may use the following links:

https://files.rcsb.org/download/1RDQ.pdb

https://files.rcsb.org/download/1E8X.pdb


To find the cavity in contact with a residue or HET group, add the argument '-a' followed by Residue Name - Residue number - Chain - Alternate location (or dash if none). For example:

```
./Get_Cleft -p ./hive/pdb/1e8x.pdb -o ./hive/clefts/1E8X -s -a ATP3000A-
```
This will create two files in the directory specified with -o :
1E8X_ATP3000A-_sph_1.pdb (cavity volume represented by spheres)
1E8X_ATP3000A-_clf_1.pdb (residues in contact with the cavity volume)

These files are used as input in mif to define where the MIF interaction vectors will be calculated.

You can calculate clefts of 1RDQ  with ATP 600 E alt. B:
```
./Get_Cleft -p ./hive/pdb/1rdq.pdb -o ./hive/clefts/1RDQ -s -a ATP600EB
```

##############################

2. Add Hydrogens to Input PDBs

##############################


Hydrogen atoms are used to calculate the directionality of Hydrogen bond (H-bond) donors and correct the placement of terminal O/N atoms for asparagine and glutamine as well as C/N for histidine as these residues can often be erroneously modeled during refinement. Use the reduce program (https://github.com/rlabduke/reduce). This can also be done with a GUI by using PyMOL.

```
./reduce -FLIP ./hive/pdb/1ex8.pdb > ./hive/pdb/1E8Xh.pdb
```
```
./reduce -FLIP ./hive/pdb/1rdq.pdb > ./hive/pdb/1RDQh.pdb
```

###################################################################

3. Calculate Molecular Interaction Fields in Both Cavities with mif

###################################################################


Use the pdb with hydrogens create by reduce (1E8Xh.pdb) and the cleft file (1E8X_ATP3000A-_sph_1.pdb). Argument -l allows you to constrain the grid around a ligand or residue up to a distance (in Angstrom) specified with -r. Argument -t is the prefix of the output file name.
```
./mif -p ./hive/pdb/1E8Xh.pdb -g ./hive/clefts/1E8X_ATP3000A-_sph_1.pdb -o ./hive/mifs/ -s 1 -l ATP3000A- -t 1E8X
```
```
./mif -p ./hive/pdb/1RDQh.pdb -g ./hive/clefts/1RDQ_ATP600EB_sph_2.pdb -o ./hive/mifs/ -s 1 -l ATP600EB -t 1RDQ
```

#################################################

4. Generate the PyMol script to visualize the MIF

#################################################


This will create a .pml file that you can open with PyMol. It will show the grids at the specified resolution (if any). It will also show colored spheres that represent position of potential interactions found below the threshold. The colors are probe specific, see below.
```
perl ./mifView.pl -m ./hive/mifs/1E8X.mif -o ./hive/mifView/
```
```
perl ./mifView.pl -m ./hive/mifs/1RDQ.mif -o ./hive/mifView/
```
hydrophobic - cyan
aromatic - orange
Hbond donor - blue
Hbond acceptor - red
Positive charge - green
Negative charge - magenta


#####################################

5. Find MIF similarities using IsoMIF

#####################################


Give the two input MIF files as input with -p1 and -p2 and the output file with -o. With argument -c, you can define in which grid resolution (0=2.0, 1=1.5, 2=1.0 and 3=0.5) you want to find mif similarities. The MIF at this resolution must have been calculated (see argument -z of mif). You can set a distance threshold with -d (in Angstroms) to determine the geometric variability between nodes in the association graph.

The example below will search mif similarities between the two query MIFs in the 1.5 Angstrom resolution grid:
```
./isomif -p1 ./hive/mifs/1E8X.mif -p2 ./hive/mifs/1RDQ.mif -o ./hive/match/ -c 2 -l1 -l1 ATP3000A- -l2 ATP600EB
```
You can also superimpose the MIFs using a list of corresponding atoms ids with the argument -c set to -2. To superimpose the MIFs based on the ATP molecules of 1E8X and 1RDQ, use argument -q to provide two lists (seperated by ' ') of corresponding ATP atoms ids (sep. by ','): -q 6833,6834,6836,6830,6828,6822,6826,6819,6824,6811 2964,2965,2967,2961,2959,2953,2957,2950,2955,2942.
```
./isoMif -p1 ./hive/mifs/1E8X.mif -p2 ./hive/mifs/1RDQ.mif -o ./hive/match/ -c -2 -q 6833,6834,6836,6830,6828,6822,6826,6819,6824,6811 2964,2965,2967,2961,2959,2953,2957,2950,2955,2942
```
The similarities using -c -2 are represented by the common potential interactions found at -d distance from each other in both MIFs after superimposition.

You can decide to identify all cliques of the association graph by specifying -s 1. By default, only the first clique is identified (-s 0). You can set -s to 1 but stop after a certain number of cliques are found, defined with the -a argument.

All the cliques identified will be printed in the output files with their score (nodes, tanimoto, RMSD of the clique, etc), the common interactions matched and their positions. To ommit printing the common interactions found in common (desired to save disk space in highthroughput) add argument '-e'.

You can also add the similarity in a suffix of the output file name ("_nodes_tanimoto") by adding '-w'.

The RMSD of a residue or bound molecule after the superimposition of the MIF similarities found in each clique can be printed in the filename (with -w) and in the output file name. Add argument '-l 1' and then specify the residue name, number, chain and alternate location (similarily with the MIF program). Don't forget to add a '-' when for example the alternative location wants to be ommited. For example:
```
./isoMif -p1 ./hive/mifs/1E8X.mif -p2 ./hive/mifs/1RDQ.mif -o ./hive/match/ -c 1 -l 1 -l1 ATP3000A- -l2 ATP600E-
```

####################################################################

6. Generate PyMol script to visualize the matched and unmatched MIFs

####################################################################

```
perl ./isoMifView.pl -m ./hive/match/1E8X_match_1RDQ.isomif -o ./hive/matchView/ -g 2
```
Argument -g defines the grid resolution of the matched nodes to view. It corresponds to argument -c of isoMif. In this PyMol file, for a given probe, to distinguish if it represents the Mif of the first or of the second protein, we use two sphere sizes. The big spheres represent the first protein (here 1E8X, in green), the small spheres the second protein (1RDQ, in cyan). Semi-transparent spheres represent the initial Mif calculated in the cavity and opaque spheres represent the subset that was found similar. In the case of 1E8X and 1RDQ there are only a few semi-transparent spheres as the two proteins are highly similar.
