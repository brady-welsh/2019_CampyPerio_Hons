## Goal
To "clean" data provided by the Australian Center of Ancient DNA (ACAD) and downloaded from the HMP database.

## Tasks
 1. Remove duplicate reads
 2. Remove Human host reads
 3. Check sequence Quality

All acquired sequencing data had already undergone trimming to remove flanking sequences such as adapters.

## Scripts
**1. Remove Duplicate Reads**

Duplicate reads provide redundant information and take up considerable memory during subsequent analysis. Removal of these sequences will ensure more accurate and memory efficient analysis.

The BBmap package was installed using Miniconda:
 `conda create -c bioconda -n BBmap bbmap`
 which contained the dedupe2 program. This program was then utilised to remove the duplicate reads present in the Modern ACAD raw sequencing data.
 
    #!/bin/bash
    #SBATCH -p highmem
    #SBATCH -N 1
    #SBATCH -n 3
    #SBATCH --time=12:00:00
    #SBATCH --mem=500GB
    
    # source activate BBmap
    
    dedupe2.sh \
    in={path to FastQ files for input} out={path to output directory} ac=f
   
   This script was then executed on the UofA's HPC; Phoenix, using `sbatch`
   #
   **2. Remove Human host reads**

Next the Human host reads were removed to ensure any Human sequences would not interfere with any subsequent alignments etc. Removal of these reads would reduce risk of misalignments leaving only microbial reads to be aligned.

KneadData is a program that is able to map all of the supplied sequences to a Human reference genome and remove any sequences that match to this reference. After installation of a Human reference genome the Modern ACAD data was ran through KneadData to remove host contamination. The program Parallel was utilised to run the command on multiple input files using only a single command.

    #!/bin/bash
    #SBATCH -p batch
    #SBATCH -N 1
    #SBATCH -n 30
    #SBATCH --time=05:00:00
    #SBATCH --mem=64GB
   
    # module load parallel
    # source activate Knead
    
    parallel -j 1 --link \
    'kneaddata -i {1} -i {2} \
    -o {path to output directory} \
    -db {path to kneaddata homo-sapien database} \
    -p 10 -t 32 --bypass-trim --remove-intermediate-output' \
    ::: {path to pair 1} ::: {path to pair 2}
This script was then executed on the UofA's HPC; Phoenix, using `sbatch`
#
**3. Check Sequence Quality**

After the raw data was successfully processed and the unwanted sequencing data had been removed, the quality of the remaining sequences required confirmation.

FastQC is a commonly used program when it comes to quality reassurance of sequencing data. The program produces .html outputs which can be opened in a web browser. These outputs display several different results from a respective quality test.

    #!/bin/bash
    #SBATCH -p batch
    #SBATCH -N 1
    #SBATCH -n 2
    #SBATCH --time=05:00:00
    #SBATCH --mem=16GB

    # Module load FastQC
   
    fastqc -t 2 \
    -o {path to Ouput Directory} {path to Input .fastq Files}
This script was then executed on the UofA's HPC; Phoenix, using `sbatch`
#
