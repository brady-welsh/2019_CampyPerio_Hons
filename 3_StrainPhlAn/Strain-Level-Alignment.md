Strain Level Alignment
======================
## Goal
To profile the levels of identifiable strains of *Campylobacter showae* to discover any change in consistantly high population seen in both healthy individuals and those with periodontal disease.

## Tasks

1. [Produce .sam files of samples]()
2. [Convert samples to markers]()
3. [Files for BLAST database]()
4. [Identify present strains using StrainPhlAn]()

## Scripts
#### Produce .sam files of samples
A MetaPhlAn conda package was installed using miniconda which included both the MetaPhlAn and StrainPhlAn programs.

To begin the strain analysis of the healthy and diseased oral caclulus samples to further understand any changes in *C. showae* populations, the data had to first be run through MetaPhlAn2 to produce the required `.sam` files. 
The following script was run on the UofA's HPC: phoenix, in order to produce the necessary files:

    for i in {input directory}/*.fastq.gz;
    do 
	    metaphlan.py $i ${i%.fastq.gz}_profile.txt \
	    --bowtie2out ${i%.fastq.gz}_bowtie2.txt \
	    --samout ${i%.fastq.gz}.sam.bz2 \
	    --input_type fastq --nproc 32;
    done
This script produced a `.sam` files for each `.fastq` sample input.
#
#### Convert samples to markers
These files had to then be converted to `.markers` files which are used by the StrainPhlAn program. these `.markers` files contain certain sequences from each read found in the `.fastq` sample files which correspond to specific marker genes for a given species. These marker genes are known loci of the bacterial genome which vary between strains of a specific taxa.
To produce these `.markers` files, the program `sample2marker` present in the MetaPhlAn conda package was used, and the subsequent script was executed:

    for i in {path to input directory}/*.sam.bz2;
    do
	    sample2markers.py --ifn_samples $i \
	    --input_type sam \
	    --output_dir {path to output directory};
    done
The resulting `.markers` files were then used for the following database production and StrainPhlAn analysis.
#
#### Files for BLAST database
Before running the markers through StrainPhlAn, a BLAST database for *Campylobacter showae* had to first be produced. Using the added database, the StrainPhlAn program would have an increased specificity for *C. showae* markers.
The first step in producing the BLAST database is to use bowtie2 to produce a `.fasta` file containing all of the markers in the MetaPhlAn2 database: mpa_v20_m200:

    bowtie2-inspect ${mpa_dir}/db_v20/mpa_v20_m200 > all_markers.fasta

Then using this `all_markers.fasta` file, the clade specific markers for *C. showae* could be extracted using the `extract_markers.py` command which is also included in the MetaPhlAn conda package:

    extract_markers.py \
	    --mpa_pkl ${mpa_dir}/db_v20/mpa_v20_m200.pkl \
	    --ifn_markers all_markers.fasta --clade s__Campylobacter_showae \
	    --ofn_markers s__Campylobacter_showae.markers.fasta

This output contains all of the *C. showae* specific markers in the MetaPhlAn2 built-in database and can then be collated with an NCBI reference genome to further the specificity of the database. This database would be built using the program `makeblastdb` which requires the BLAST+ tools package (downloaded using miniconda: `conda install blast -c bioconda`). This tool runs simultaneously with the StrainPhlAn program provided each of the required files is supplied.

#
#### Identify present strains using StrainPhlAn
Following all of the prerequisite steps the files produced from aforementioned scripts were then put through the `strainphlan.py` program to perform the strain-level alignment specific for *Campylobacter showae*

    strainphlan.py \
	    --ifn_samples {path to input}/*.markers \
		--ifn_markers {path to showae markers}/s_Campylobacter_showae-v293CHOCOPhlAn.markers.fasta \
		--ifn_ref_genomes {path to showae reference genome}/Campylobacter-showae.fna.gz \
		--output_dir {path to output directory} \
		--clades s_Campylobacter_showae --nprocs_main 16
		--relaxed_parameters2

This produced several .tree and .fasta alignment files which could be used for the bootstrapping and visualisation with RAxML and Figtree
#
#### Boostrapping and Tree Production
Finally, the fasta alignment files from the StrainPhlAn strain-level alignment were then ran through the following RAxML script to calculate bootstrapping support for the alignments and produce .tree files which could be opened and visualised in FigTree (v1.1.4)

RAxML was loaded on the HPC using `module load RAxML`

    raxmlHPC-HYBRID-SSE3 \
	    -f a \
	    --threads 16 \
	    -x 2606 \
	    -p 2606 \
	    -# autoMRE \
	    -m GTRGAMMA \
	    -s /localscratch/bwelsh/4_PocketHealthy/2_StrainPhlAn/s__Campylobacter_showae.fasta -n RAxML_Both

The susequent tree files were then opned in FigTree for analysis of strain presence and diversity.
#
