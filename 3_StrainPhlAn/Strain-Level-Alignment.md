Strain Level Alignment
======================
## Goal
To profile the levels of identifiable strains of *Campylobacter showae* to discover any change in consistantly high population seen in both healthy individuals and those with periodontal disease.

## Tasks

1. [Produce .sam files of samples]()
2. [Convert samples to markers]()
3. [Produce a BLAST database]()
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
#### Produce a BLAST database

#
#### Identify present strains using StrainPhlAn