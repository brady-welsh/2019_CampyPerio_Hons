Processing HMP Data
===================

## Goal
To "clean" data downloaded from the Human Microbiome Project (HMP) database.

## Tasks
 1. [Select random sample of sequences](https://github.com/brady-welsh/campy-perio/blob/master/1_Processing/Processing-HMP-Data.md#1-select-random-sample-of-sequences)
 2. [Remove Human host reads](https://github.com/brady-welsh/campy-perio/blob/master/1_Processing/Processing-HMP-Data.md#2-remove-human-host-reads)
 3. [Remove duplicate reads](https://github.com/brady-welsh/campy-perio/blob/master/1_Processing/Processing-HMP-Data.md#3-remove-duplicate-reads)
 4. [Remove low complexity reads](https://github.com/brady-welsh/campy-perio/blob/master/1_Processing/Processing-HMP-Data.md#4-remove-low-complexity-reads)
 5. [Check sequence quality](https://github.com/brady-welsh/campy-perio/blob/master/1_Processing/Processing-HMP-Data.md#5-check-sequence-quality)
 
 All acquired sequencing data had already undergone trimming to remove flanking sequences such as adapters.
 
## Scripts
#### 1. Select Random Sample of Sequences
As the FastQ files obtained from the HMP database were so deeply sequenced and contained so many reads, these files had to be reduced in order to efficiently analyse the data. The program Seqtk is able to generate new FastQ files with a random selection of sequences from the original, larger FastQs, as well as being able to generate a seed for the selection for reproducablity purposes.
Seqtk was a module already loaded on the HPC; Phoenix, and therefore was loaded using `module load seqtk`.

    #!/bin/bash
    #SBATCH -p highmem
    #SBATCH -N 1
    #SBATCH -n 16
    #SBATCH --time=5:00:00
    #SBATCH --mem=50GB
    
    # module load seqtk
    
    for i in (path to input .fastq files);
    do
      seqtk sample -s2606 \
      $i 2000000 > ${i/.fastq/-2Mss.fastq};
    done
This script was then executed on the UofA's HPC; Phoenix, using `sbatch`

*Program Versions:*

seqtk/1.3-foss-2016b
#
#### 2. Remove Human Host Reads
Next the Human host reads were removed to ensure any Human sequences would not interfere with any subsequent alignments etc. Removal of these reads would reduce risk of misalignments leaving only microbial reads to be aligned.

KneadData is a program that is able to map all of the supplied sequences to a Human reference genome and remove any sequences that match to this reference conda create -n Knead kneaddata p=3. After downloading a Human reference genome the HMP data was ran through KneadData to remove host contamination. The program Parallel was utilised to run the command on multiple input files using only a single command.

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
#### 3. Remove Duplicate Reads
Duplicate reads provide redundant information and take up considerable memory during subsequent analysis. Removal of these sequences will ensure more accurate and memory efficient analysis.

The BBmap package was installed using Miniconda:
`conda create -c bioconda -n BBmap bbmap`
which contained the dedupe2 program. This program was then utilised to remove the duplicate reads present in the HMP raw sequencing data.

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
#### 4. Remove Low Complexity Reads
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
#### 5. Check Sequence Quality
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
