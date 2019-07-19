MALT Alignment
=============
## Goal
To align sequences from processed data with with a reference oral microbiome and begin calculating the proportion of *Campylobacter* reads sequenced.

## Tasks

 1. [Run data through MALT](https://github.com/brady-welsh/campy-perio/blob/master/2_MALT-Alignment.md#2-run-data-through-malt)
 2. [Analysis in MEGAN](https://github.com/brady-welsh/campy-perio/blob/master/2_MALT-Alignment.md#3-analysis-in-megan)

Data used in these analysis is the final output from the "1_Processing-Data" section of this notebook and information of indices used for the alignment can be found in the "Indices" folder.

## Scripts
#### 1. Run data through MALT

In order to investigate the proportion of *Campylobacter* spp. in the sequenced oral microbiomes, they must be aligned to the reference indices produced using `malt-build`. The second command in the MALT package `malt-run` will align the processed data to the reference index and determine which species each sequence belongs to. This script will be twice, using one of the two MALT indices produced per alignment.

    #!/bin/bash
    #SBATCH -p highmem
    #SBATCH -N 1
    #SBATCH -n 32
    #SBATCH --time=30:00:00
    #SBATCH --mem=1500GB
   
    # module load malt
    
    malt-run -i (path to input fastq files) \
    -a (path to output directory) \
    -ou (path to output directory) \
    -at SemiGlobal -f Text -d (path to directory containing MALT index) \
    -mem load --mode BlastN -t 32 -v
This script was then executed on the UofA's HPC; Phoenix, using `sbatch`

Output from this script will be in .blastn format.
#
## Analysis
#### 2. Analysis in MEGAN

As the output from MALT comes in a .blastn format, the files must be converted to .rma6 format in order to be read by MEGAN. The blast2rma command comes in the MALT package and can be used for this conversion.

To start all .blastn and processed .fastq files were moved to the same directory.
Parallel was used to run the command on each file.

    #!/bin/bash
    #SBATCH -p highmem
    #SBATCH -N 1
    #SBATCH -n 32
    #SBATCH --time=14:00:00
    #SBATCH --mem=500GB
   
    # module load parallel
    # module load malt
    
    for i in (path to .blastn & .fastq files);
    do 
       b=${i%.fa*}.blastn.gz; echo blast2rma -r $i -i $b \
       -o (path to output directory) \
       -ms 44 -me 0.01 -f BlastText -alg weighted -lcp 80; \
    done | parallel -j28 --delay 6

This script was then executed on the UofA's HPC; Phoenix, using `sbatch`

These .rma6 files were then moved to a MEGAN server and opened in the MEGAN GUI. In MEGAN the alignments can be visualised and the proportions of each taxa present in the samples can be seen.
