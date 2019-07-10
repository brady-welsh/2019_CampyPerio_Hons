MALT Alignment
=============
## Goal
To align sequences from processed data with with a reference oral microbiome and begin calculating the proportion of *Campylobacter* reads sequenced.

## Tasks

 1. [Produce a MALT index](https://github.com/brady-welsh/campy-perio/blob/master/2_MALT-Alignment.md#1-produce-a-malt-index)
 2. Run data through MALT
 3. Analysis in MEGAN

Data used in these analysis is the final output from the "1_Processing-Data" section of this notebook.

## Scripts
#### 1. Produce a MALT index

MALT is able to align the input data to reference genomes in the form of an index. This index can contain multiple genomes such as the HOMD oral microbiome database, however, more sequences can be added to increase the specificity of the alignment. In this case a reference genome for each known species of oral *Campylobacter* will be added along with the HOMD database to produce a *Campylobacter* specific index.

The MALT package comes with two main components: `malt-build` for producing indicies and `malt-run` for running the alignment. MALT was already a module available on Phoenix and therefore installation was not necessary and instead only the 
`module load malt` command was needed. Databases for the index were downloaded from the *e*HOMD website and NCBI. A whole oral microbiome reference metagenome was obtained from *e*HOMD and a whole genome reference for each oral *Campylobacter* species was downloaded from NCBI. These genomes were then ran through malt-build.

    #!/bin/bash
    #SBATCH -p highmem
    #SBATCH -N 1
    #SBATCH -n 30
    #SBATCH --time=16:00:00
    #SBATCH --mem=480GB
   
    # module load malt
    
    malt-build -i (path to reference .fasta files) \
    -d (path to output directory) \
    -a2taxonomy (path to MALT taxonomy file) \
    --sequenceType DNA -t 30 -v
This script was then executed on the UofA's HPC; Phoenix, using `sbatch`
#
#### 2. Run data through MALT

In order to investigate the proportion of *Campylobacter* spp. in the sequenced oral microbiomes, they must be aligned to the reference index produced in the previous step. The second command in the MALT package `malt-run` will align the processed data to the reference index and determine which species each sequence belongs to.

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
    -at SemiGlobal -f Text -d (path to directory containing matrix) \
    -mem load --mode BlastN -t 32 -v
This script was then executed on the UofA's HPC; Phoenix, using `sbatch`

Output from this script will be in .blastn format.
## Analysis
#### 3. Analysis in MEGAN

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

These .rma6 files were then moved to a MEGAN server and opened in the MEGAN GUI.
