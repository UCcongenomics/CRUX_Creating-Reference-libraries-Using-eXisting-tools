# CRUX: Creating Reference libraries Using eXisting tools

### CRUX_release_V1		09-01-2017
#### Written by Emily Curd (eecurd@g.ucla.edu) and Gaurav Kandlikar (gkandlikar@ucla.edu)
#### Developed at UCLA for the University of California Conservation Consortium's CALeDNA Program

## Introduction
  
Taxonomic assignments of metabarcoding reads requires a comprehensive reference library. For 16S and 18S metabarcode reads, we use the Silva high quality ribosomal databases (Glöckner et al., 2017) for reference libraries. For all other metabarcodes we construct custom reference libraries using our in-house wrapper script, CRUX: Constructing Reference libraries Using eXisting tools. CRUX generates reference libraries by running in silico PCR (ecoPCR; Ficetola et al., 2010) against the EMBL standard nucleotide sequence database (Stoesser et al., 2002) to generate a seed library of reads with unique taxon identifiers. Because many sequencing records are deposited to Genbank (Benson et al., 2012) with the primer regions removed from the read, we BLAST (Camacho et al., 2009) the seed library against the NCBI nucleotide blast database (ftp://ftp.ncbi.nlm.nih.gov/blast/). Blast results are de-replicated by version accession number and converted to fasta format. A taxonomy identification file is generated from the fasta formatted blast output using entrez-qiime (https://github.com/bakerccm/entrez_qiime), the NCBI’s taxonomy dump and map of association between taxonomy and accession version numbers (ftp://ftp.ncbi.nlm.nih.gov/pub/taxonomy/). To minimize the reference library file size and merge highly similar reads, we use Qiime (Caporaso et al., 2010) to cluster the reference library at 99% sequence similarity using Usearch (Edgar, 2010) and generate a corresponding taxonomy file assign taxonomy in Qiime using Uclust (Edgar, 2010) at 97%. 

Next steps: The reference library can then be used to generate a multiple sequence alignment using muscle (ref) and a phylogenetic tree using (decide). 

## Before CRUX is operable, you will need to download, install and/or build several programs and databases. 
**__First Download the crux_release_V1_db folder.__** The executables and database folders should be accessible from this folder. However, if you already have any these programs or databases, there is no need to add them to the crux_release_V1_db folder. Instead update the file paths or loading commands to the Crux_config.sh and crux_vars.sh folders accordingly.

**__Programs__**


1. OBItools: https://git.metabarcoding.org/obitools/obitools/wikis/home
	* OBItools is used to generate reference libraries for the ecoPCR in silico PCR step for CRUX.
	* OBItools does not need to be installed in the crux_release_V1_db folder.
	* Installation information can be found here: http://metabarcoding.org//obitools/doc/welcome.html#installing-the-obitools

2. ecoPCR: https://git.metabarcoding.org/obitools/ecopcr/wikis/home
	* If you are not modifying the Crux_config.sh, then the path to the ecoPCR executable should be as follows: ~/crux_release_V1_db/ecoPCR/src/ecoPCR

3. BLAST+: https://www.ncbi.nlm.nih.gov/books/NBK279690/
	* the lastest BLAST executables can be downloaded from: ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.6.0/
	* If you are not modifying the Crux_config.sh, then the path to the blastn executable should be as follows: ~/crux_release_V1_db/ncbi-blast-2.6.0+/bin/blastn

4. entrez_qiime: https://github.com/bakerccm/entrez_qiime
	* **entrez_qiime.py** is already included in crux_release_V1_db folder

5. Qiime 1: http://qiime.org/index.html
	* Qiime 1 does not need to be installed in the crux_release_V1_db folder, however you will need to verify that the Crux_config.sh is modified for you computing environment. 
	* Installation information can be found here: http://qiime.org/install/install.html
	* We will transition to Qiime 2 by January 01, 2018. 
  
  
**__Databases to download__**

1. NCBI taxonomy dump: ftp://ftp.ncbi.nlm.nih.gov/pub/taxonomy/taxdump.tar.gz
	* If you are not modifying the Crux_config.sh, then the path to the taxonomy folder should be as follows: ~/crux_release_V1_db/TAXO 
	* The folder should contain the following files: delnodes.dmp, merged.dmp, names.dmp, nodes.dmp
	* download information can be found here: https://github.com/bakerccm/entrez_qiime/blob/master/entrez_qiime.pdf

2. NCBI accession2taxonomy file:  ftp://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/nucl_gb.accession2taxid.gz
	* If you are not modifying the Crux_config.sh, then the path to the accession to taxonomy file should be as follows: ~/crux_release_V1_db/accession2taxonomy/nucl_gb.accession2taxid
	* download information can be found here: https://github.com/bakerccm/entrez_qiime/blob/master/entrez_qiime.pdf
	
3. NCBI BLAST nt library: ftp://ftp.ncbi.nlm.nih.gov/blast/db/
	* mkdir /NCBI_blast_nt in ~/crux_release_V1_db
	* cd ~/crux_release_V1_db/NCBI_blast_nt
	* wget ftp://ftp.ncbi.nlm.nih.gov/blast/db/nt*
	* gunzip tar -zxvf *.tar.gz

4. EMBL_<date>_std database files: ftp://ftp.ebi.ac.uk/pub/databases/embl/release/std
	* Determine which EMBL databases you wish to include.  
		* One strategy is to download all of the files for only some of the organism: fun, inv, mam, phg, pln, pro, rod, syn, tgn, vrl, vrt 
		* It is ok to skipped hum, and mus because those reads will be picked up while blasting.
	* These do not need to be stored in crux_release_V1_db, but they need to be stored as a single folder or as a series of folders based on taxonomic groupings (e.g. store all prokaryotes (pro) in a single and separate folder, store all plant (pln) in a single and separate folder, etc.)
	* These are very large files, and it is less memory intensive to download them in small batches, and convert them into many small obitools/ecopcr readable databases.
		* wget ftp://ftp.ebi.ac.uk/pub/databases/embl/release/std/rel_std_fun*
		* wget ftp://ftp.ebi.ac.uk/pub/databases/embl/release/std/rel_std_pro*, 
		* etc...  
		* gunzip *.gz
  
 **__Building OBItools/ecoPCR Readable Databases__**  
 Convert embl databases into obitools/ecopcr readable databases using obiconvert
* the obiconvert python script is part of the OBItools.
	* the documentation can be found here: http://metabarcoding.org/obitools/doc/scripts/obiconvert.html
	* a tutorial can be found here: http://metabarcoding.org/obitools/doc/wolves.html
* The obiconvert command requires:
	* the path to the ncbi taxonomy folder (-t)
		*downloaded above ~/crux_release_V1_db/TAXO
	* the file format (--embl)
	* the output folder path (--ecopcrdb-output) 
		* the file path needs to be /crux_release_V1_db/Obitools_databases/some_folder_name_that_corresponds_to_the_type_of embl_seqeunces 
			* e.g. ~/crux_release_V1_db/Obitools_databases/EMBL_6167017_std_pro
		* the input file(s) path(s) 
			* e.g. the folder where all prokaryote (pro) files are stored in a single and separate folder
* Depending on the size of the EMBL database files this may take a lot of time and a lot of RAM
* The command is as follows:
	* obiconvert -t </path/to/taxonfile> --embl --ecopcrdb-output=/path/to/output /path/to/inputs --skip-on-error
		* e.g. obiconvert -t ~/crux_release_V1_db/TAXO --embl --ecopcrdb-output= ~/crux_release_V1_db/Obitools_databases/EMBL_6167017_std_pro ~/EMBL_pro/*.dat
  
  
## Running CRUX
* The command to run CRUX is:
	* sh crux_release_V1.sh -n metabarcode_target_name -f forward_primer_sequence -r reverse_primer_sequence -d path_to_CRUX_db_folder -o path_to_output_folder
* An example:
	* sh ~/crux_release_V1_db/crux_release_V1.sh -n CO1 -f GGWACWGGWTGAACWGTWTAYCCYCC  -r TANACYTCnGGRTGNCCRAARAAYCA -d ~/crux_release_V1_db -o ~/crux_release_V1_db/CO1 
* The final database files can be found in the metabarcode_target_name_database folder with in the user specified output folder. 
	
	
## Step by Step Explanation of CRUX

Part 1: ecoPCR

1. Run ecoPCR using the user specified primer on the user generated OBItools compatible embl databases.  
	* ecoPCR parameters can be altered in the /crux_release_V1_db/crux_vars.sh file
2. ecoPCR results are de-replicated based on taxon id (taxid), and converted to fasta format.
	* The resulting fasta files are segregated by the number of database files generated by the user. 
	
Part 2: blasting

1. The fasta files are blasted against the the NCBI nucleotide blast databases using blastn. 
	* blastn parameters can be altered in the /crux_release_V1_db/crux_vars.sh file
2. The blast results are de-replicated by NCBI accession version number and converted into fasta format.
3. entrez-qiime is then used to determine taxonomy for each read based on NCBI version accession number. 
4. **pending**
	* for all reads where the taxonomy has NA;NA;NA at the beginning of the taxonomic path, remove those reads from the cleaned blast results fasta output and corresponding taxonomy file.
	
Part 3: Clustering

1. Reduce the dataset to read clusters with 99% identity using usearch, implemented in Qiime
	* The user can change the identity threshold in the /crux_release_V1_db/crux_vars.sh file
	* Clustering at 99% reduces the size of the dataset and collapses reads that are identical and nearly identical.
2. Assign taxonomy at 97% using uclust, implemented in Qiime
	* The user can change the identity threshold in the /crux_release_V1_db/crux_vars.sh file
	* Assigning taxonomy at 97% reduces the number of unassignable reads in the resulting database
		* When there are several species with in the same genus, assigning reads at 99% generates many unassigned reads, however when clustered at 97% the reads formerly unassigned are typically correctly assigned to genus.  
		
Part 4: **pending**
* multiple sequence alignment
* phylogenetic tree 
* detailed log files
* summary statistic files
  
  
  
  
  
  
  
  
## References:

Benson, D.A., Cavanaugh, M., Clark, K., Karsch-Mizrachi, I., Lipman, D.J., Ostell, J. and Sayers, E.W., 2012. GenBank. Nucleic acids research, 41(D1), pp.D36-D42.

Camacho, C., Coulouris, G., Avagyan, V., Ma, N., Papadopoulos, J., Bealer, K. and Madden, T.L., 2009. BLAST+: architecture and applications. BMC bioinformatics, 10(1), p.421.

Caporaso, J.G., Kuczynski, J., Stombaugh, J., Bittinger, K., Bushman, F.D., Costello, E.K., Fierer, N., Peña, A.G., Goodrich, J.K., Gordon, J.I. and Huttley, G.A., 2010. QIIME allows analysis of high-throughput community sequencing data. Nature methods, 7(5), pp.335-336.

Edgar, R.C., 2010. Search and clustering orders of magnitude faster than BLAST. Bioinformatics, 26(19), pp.2460-2461.

Ficetola, G.F., Coissac, E., Zundel, S., Riaz, T., Shehzad, W., Bessière, J., Taberlet, P. and Pompanon, F., 2010. An in silico approach for the evaluation of DNA barcodes. BMC genomics, 11(1), p.434.

Glöckner, F.O., Yilmaz, P., Quast, C., Gerken, J., Beccati, A., Ciuprina, A., Bruns, G., Yarza, P., Peplies, J., Westram, R. and Ludwig, W., 2017. 25 years of serving the community with ribosomal RNA gene reference databases and tools. Journal of Biotechnology.

Stoesser, G., Baker, W., van den Broek, A., Camon, E., Garcia-Pastor, M., Kanz, C., Kulikova, T., Leinonen, R., Lin, Q., Lombard, V. and Lopez, R., 2002. The EMBL nucleotide sequence database. Nucleic acids research, 30(1), pp.21-26.
