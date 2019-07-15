Processing Modern ACAD Data
===========================

## Goal
To "clean" data provided by the Australian Center of Ancient DNA (ACAD)

## Tasks
 1. [Remove Human host reads](https://github.com/brady-welsh/campy-perio/blob/master/1_Processing-ACAD-Data.md#1-remove-human-host-reads)
 2. [Remove duplicate reads](https://github.com/brady-welsh/campy-perio/blob/master/1_Processing-ACAD-Data.md#2-remove-duplicate-reads)
 3. [Remove low complexity reads](https://github.com/brady-welsh/campy-perio/blob/master/1_Processing-ACAD-Data.md#3-remove-low-complexity-reads)
 4. [Check sequence Quality](https://github.com/brady-welsh/campy-perio/blob/master/1_Processing-ACAD-Data.md#4-check-sequence-quality)

All acquired sequencing data had already undergone trimming to remove flanking sequences such as adapters.

## Scripts
#### 1. Remove Human host reads

Next the Human host reads were removed to ensure any Human sequences would not interfere with any subsequent alignments etc. Removal of these reads would reduce risk of misalignments leaving only microbial reads to be aligned.

KneadData is a program that is able to map all of the supplied sequences to a Human reference genome and remove any sequences that match to this reference `conda create -n Knead kneaddata p=3`. After downloading a Human reference genome the Modern ACAD data was ran through KneadData to remove host contamination. The program Parallel was utilised to run the command on multiple input files using only a single command.

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
       -o (path to output directory) \
       -db (path to kneaddata homo-sapien database) \
       -p 10 -t 32 --bypass-trim --remove-intermediate-output' \
    ::: (path to pair 1) ::: (path to pair 2)
This script was then executed on the UofA's HPC; Phoenix, using `sbatch`


*Program Versions:*

parallel/20180922-foss-2016b

kneaddata v0.6.1
#
#### 2. Remove Duplicate Reads

Duplicate reads provide redundant information and take up considerable memory during subsequent analysis. Removal of these sequences will ensure more accurate and memory efficient analysis.

The BBmap package was installed using Miniconda:
`conda create -c bioconda -n BBmap bbmap`
which contained the dedupe2 program. This program was then utilised to remove the duplicate reads present in the Modern ACAD raw sequencing data.

    #SBATCH -p highmem
    #SBATCH -N 1
    #SBATCH -n 16
    #SBATCH --time=8:00:00
    #SBATCH --mem=250GB
    
    # source activate BBmap
    # run from directory containing .fastq files
    
    for i in *.fastq.gz;
    do 
       dedupe2.sh in=$i \
       out=(path to output directory)/${i%.fa*}_deduped.fastq.gz \
       ac=f; 
    done

This script was then executed on the UofA's HPC; Phoenix, using `sbatch`


*Program Versions:*

BBMap/36.62-intel-2017.01-Java-1.8.0_121
#
#### 3. Remove Low Complexity Reads
Reads found within the raw sequencing data can cause errors in subsequent analysis such as the alignments. In order to remove these reads the pogram Komplexity can be used. The program was downloaded from the Anaconda cloud using miniconda `conda create -c eclarke -n Komplex komplexity`
The following script was then used to remove the low complexity reads.

    #!/bin/bash
    #SBATCH -p highmem
    #SBATCH -N 1
    #SBATCH -n 16
    #SBATCH --time=10:00:00
    #SBATCH --mem=100GB
    
    # source activate Komplex
    
    for fq in (Path to input directory)/*.fastq;
    do
        kz --fasta --filter < $fq > ${fq/.fastq/-kz.fastq};
    done

This script was then executed on the UofA's HPC; Phoenix, using `sbatch`

*Program Versions:*

komplexity-0.3.6-musl
#
#### 4. Check Sequence Quality

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
    -o (path to Ouput Directory) (path to Input .fastq Files)
This script was then executed on the UofA's HPC; Phoenix, using `sbatch`


*Program Versions:*

fastqc/0.11.4
#
