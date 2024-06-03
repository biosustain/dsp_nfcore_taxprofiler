# dsp_nfcore_taxprofiler
## Taxonomical profiling using nf-core/taxprofiler pipeline

nf-core/taxprofiler is a bioinformatics best-practice analysis pipeline for taxonomic classification and profiling of shotgun short- and long-read metagenomic data. It allows for in-parallel taxonomic identification of reads or taxonomic abundance estimation with multiple classification and profiling tools against multiple databases, and produces standardised output tables for facilitating results comparison between different tools and databases.

You can find a more exhaustive description and running instructions in here:
https://nf-co.re/taxprofiler/1.1.7

Here we provide with a small manual to how to prepare, for running the pipeline and running it in the Microsoft Azure environment.

## Create samplesheet.csv as:
```
sample,run_accession,instrument_platform,fastq_1,fastq_2,fasta
2612,run1,ILLUMINA,2612_run1_R1.fq.gz,,
2612,run2,ILLUMINA,2612_run2_R1.fq.gz,,
2612,run3,ILLUMINA,2612_run3_R1.fq.gz,2612_run3_R2.fq.gz,
...
```

## Create a databases.csv sheet as:
```
tool,db_name,db_params,db_path
metaphlan,db1,,az://orange/databases/metaphlan_db
motu,db2,,az://orange/databases/db_mOTU
...
```

The databases will be store in the corresponding data lake folder called databases. Until then you have to download and prepare the databases yourself.

Files for Metaphlan were download from:
http://cmprod1.cibio.unitn.it/databases/MetaPhlAn/metaphlan_databases/

For mOTUs:
Needed to prepare the mOTUs database as follows:

```
conda create --name motus
conda activate motus
conda install -c bioconda motus
motus downloadDB
```
It got copied the database locally in here: /Users/apca/anaconda3/envs/motus/lib/python3.9/site-packages/motus/db_mOTU and I passed this dir to databases.csv

## Command line to run it using Metaphlan and mOTU
```
nextflow run nf-core/taxprofiler -profile az_test,docker --input samplesheet.csv --databases databases.csv --outdir results --perform_shortread_qc --shortread_qc_tool adapterremoval --perform_shortread_complexityfilter --perform_shortread_hostremoval --hostremoval_reference az://masldmice/host_genome/GCF_000001635.27_GRCm39_genomic.fna --run_metaphlan --run_motus --motus_use_relative_abundance --run_krona -w az://taxprofiler -resume -with-tower
```

## Seqera platform
Notice that one parameter tells the pipeline to be monitored by Seqera platform (-with-tower)
To do that login to Seqera platform and create a token (User tokens) clicking on the User settings button in the upper-right corner. 
Once created, copy it and export it in your terminal: 
```
export TOWER_ACCESS_TOKEN=<your token>
```

## Needed to change vmType that in the nextflow.config:
To meet the requirement of 12 CPUs and 72 GB of memory, we used “Standard_E16s_v3” with 16 cpus and 128GB memory
```
pools {
    auto {
        autoScale = true
        vmType = 'Standard_E16s_v3'
        }
    }
```
