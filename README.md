# De-Novo-Genome-Sequencing-project
This is the working repository for the genome sequencing assembly

## Overall
This projects aim is to assemble HiFi Pacbio reads from the yeast **Saccharomyces cerevisiae** and apply quality assesments. The overall workflow looks like:
- Quality assessment using fastqc
- de novo assembly
- Quast quality assessment
- BUSCO assessment

## Dataset
The dataset is a HiFi Pacbio reads from the yeast Saccharomyces cerevisiae.
The data can be downloaded from NCBI Sequence Read Archive (SRA)

Download the reads for the following page or use sra-tools:
https://trace.ncbi.nlm.nih.gov/Traces?run=SRR13577846

## Installation and Setup
For the workflow, it is advisible to create and use a new conda environment. Conda is a open source management system and can be downloaded on the website https://docs.conda.io/en/latest/. Normally it comes with Anaconda or Miniconda. Conda was installed and used in the version conda 22.11.1.

Set up a new directory and environment
```
mkdir DeNovoProj
conda create -n DeNovoProj
conda activate DeNovoProj
```
Now every package and software that is downloaded will be in that specific environment. For the quality checking, de novo genome assembly and assesments, a few prior packages need to be downloaded and installed. This is described in every section where the software is needed.

# Quality assessment using fastqc
To get a overview of the general quality of the sequenced read, FastQC v0.11.9 is installed with conda and used in the terminal.
```
conda install -c bioconda fastqc
```

The qualitiy assesment is run with the fastq file `fastqc SRR13577846.fastq`, producing a html overview of different qualitiy measurements. 
From the the quality assesment it can be seen that the per base sequence content and per sequence GC content triggers an error at the fastqc and sequence length destribution an error.

Since the quality of the reads looks in overall good, the assembly can be continued with the current data, without further trimming needed.

## [OPTIONAL] Data trimming using trimmomatic

## De Novo assembly

In the de novo assembly workflow the tool pb-assembly with the FALCON assembler is unfortunately not working, eventough it would be advisible to use this pacbio long read specific assembler for optimal assembly. Specific as it says on the pb-assembly github page, "pb-assembly is the bioconda recipe encompassing all code and dependencies necessary to run" (https://github.com/PacificBiosciences/pb-assembly). The pb-assembly is stilled in the workflow and will be added on a later commit.

Since the pb-assembler seemed to be not working right now, it was continued with the hifiasm assembler and peregrine-2021 assembler. For that the hifiasm was installed via conda and run like, where -o specifies the output namings and -t the number of CPUs used.
```
conda install -c bioconda hifiasm
hifiasm -o SRR135.asm -t 6 SRR13577846.fastq
```

The other as seemed better assembler used is Peregrine. Peregringe is a really fast and modern genome assembler, with easy to use commands. It was installed using bioconda in the version 0.4.13 and can be retrieved from https://github.com/cschin/peregrine-2021.

Before the assembly with peregrine can bes started, the program requires to put the paths to the sequence read files (.../SRR13577846.fastq.gz) in a text file, e.g. called `seq.lst`. So that peregrine can find the read data. Also a output directory `asm_out` is created, in where the assembly data will be stored.

To start the assembly, the following command need to be run:
```
conda install -c bioconda peregrine-2021
pg_asm seq.lst asm_out 
```

Peregrine should be really fast and produce multiple contigs fasta files and additional read and assembly files. The main contig is stored in the file, containing an m in it, this is the file that will be further analysed. Optional the --fast flag can be added, when running the assembly, resulting in a quicker assembly, but with a lower quality.


## Quast quality assessment
QUAST is the Quality Assessment Tool for Genome Assemblies, which valuates genome assemblies and is offered on https://quast.sourceforge.net. It provides different graphical and non graphical quality statistics and can be run as the following, where the .py script of quast is executed on the assembly fasta output files (the number of threads has to be set for some computers).
```
conda install -c bioconda quast
./quast.py ../denovo_asm/denovo_peregr/asm_out/asm_ctgs_* -t 1
```

Quast will report some statistical reports with graphical interactive information through a webpage, this can be further analysed to evaluate the quality of the assemblies as done here.

Looking in the statistics, it is possible to compare the peregrine and hifiasm assemblies:
| Assembler | #contigs | Largest contig | Total length | N50    | Missasemblies |
|-----------|----------|----------------|--------------|--------|---------------|
| hifiasm   | 49       | 1506376        | 12730920     | 809047 | 119           |
| peregrine | 22       | 1505876        | 12307599     | 920022 | 0             |

It can be seen that the number of contigs and missmatches differs a lot. Peregrine generally did a better assembly with less or even 0 missmatches, but has a overall higher N50. 

## BUSCO assesment
BUSCO (Benchmarking Universal Single-Copy Orthologs) is a bioinformatics tool that evaluates the completeness of genome assemblies and gene sets. It accomplishes this by searching for a set of genes that are present in a single copy in most species, comparing them to a reference dataset of known orthologs, and reporting the percentage of complete, fragmented, and missing orthologs in the input data. BUSCO is widely used in genomics research to assess the quality of genomic data, compare different assemblies or gene sets, and identify potential missing genes. (https://busco.ezlab.org)

The installation and run of busco can be tricky, which is why it's advisable to use the conda installation and the here provided command.
```
# install busco
conda install -c conda-forge -c bioconda busco  
# run busco with auto lineage to find the best lineage
busco -m genome -i ../denovo_asm/denovo_peregr/asm_out/asm_ctgs_m.fa -o busco_out --auto-lineage -f
```

For BUSCO the mode (-m) has to specified as genome, then the input fasta contig assembly provided (here for peregrine), the output folder where the results are saved and a lineage (either set manually with -l to eukaryotic (because of yeast) or using the option to let busco find the best lineage with --auto-lineage), as well as -f, forcing the run to rewrite prior failed runs for exampel.

Busco will provide a summary, simmilar to:
```
# BUSCO version is: 5.4.4 
# The lineage dataset is: saccharomycetes_odb10 (Creation date: 2020-08-05, number of genomes: 76, number of BUSCOs: 2137)
# Summarized benchmarking in BUSCO notation for file /Users/dominiquefastus/DeNovoProj/denovo_asm/denovo_peregr/asm_out/asm_ctgs_m.fa
# BUSCO was run in mode: euk_genome_met
# Gene predictor used: metaeuk

	***** Results: *****

	C:99.6%[S:97.1%,D:2.5%],F:0.1%,M:0.3%,n:2137	   
	2128	Complete BUSCOs (C)			   
	2075	Complete and single-copy BUSCOs (S)	   
	53	Complete and duplicated BUSCOs (D)	   
	3	Fragmented BUSCOs (F)			   
	6	Missing BUSCOs (M)			   
	2137	Total BUSCO groups searched		   

Assembly Statistics:
	22	Number of scaffolds
	22	Number of contigs
	12307599	Total length
	0.000%	Percent gaps
	920 KB	Scaffold N50
	920 KB	Contigs N50
```
The results for the number of complete BUSCOs will be similar among all assemblies, when the same lineage database, which it should be compared to is selected.

## Sum up
The de novo assembly is a good strategie and there are many good tools, which will produce good genome assemblies. Therefore it mainly depends on the data (how large will the genome be?) and the computing circumstances (time, money and computer efficiency?).
