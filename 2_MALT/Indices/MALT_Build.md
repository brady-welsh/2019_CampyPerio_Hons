#### Produce a MALT index

MALT is able to align the input data to reference genomes in the form of an index. This index can contain multiple genomes such as the HOMD oral microbiome database, however, more sequences can be added to increase the specificity of the alignment. In this case a reference genome for each known species of oral *Campylobacter* will be added along with the HOMD database to produce a *Campylobacter* specific index.

The MALT package comes with two main components: `malt-build` for producing indicies and `malt-run` for running the alignment. MALT was already a module available on Phoenix and therefore installation was not necessary and instead only the 
`module load malt` command was needed. Databases for the index were downloaded from the *e*HOMD website and NCBI. A whole oral microbiome reference metagenome was obtained from *e*HOMD and as well as 247 *Campylobacter* complete reference sequences downloaded from NCBI.

247 complete *Campylobacter* reference sequences were downloaded from NCBI using `ncbi-genome-download` installed using `conda` from the Anaconda cloud. Script run to download refseqs: 

`ncbi-genome-download --format fasta --genus Campylobacter bacteria --assembly-level complete`

The reference sequences (listed in Campylobacter-complete-refseqs.txt) where compiled in the same directory to be ran through `malt-build` using the following script:

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

Another MALT index was produced using complete and incomplete *Campylobacter* assemblies downloaded again using `ncbi-genome-download`. These sequences were downloaded using the following script:

`ncbi-genome-download --format fasta --genus Campylobacter bacteria`

2927 *Campylobacter* genomes were downloaded (listed in Campylobacter-All-refseqs.txt) and compiled into a directory with the HOMD oral-microbiome database. The `malt-build` script above was then run using these sequences to align to the modern data.

*Program Versions:*

MALT

ncbi-genome-download
