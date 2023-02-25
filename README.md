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

The qualitiy assesment is run with the fastq file `fastqc SRR13577846.fastq, producing a html overview of different qualitiy measurements. 
From the the quality assesment it can be seen that the per base sequence content and per sequence GC content triggers an error at the fastqc and sequence length destribution an error.

Since the quality of the reads looks in overall good, the assembly can be continued with the current data, without further trimming needed.

## [OPTIONAL] Data trimming using trimmomatic

## De Novo assembly

In the de novo assembly workflow the tool pb-assembly with the FALCON assembler is unfortunately not working, eventough it would be advisible to use this pacbio long read specific assembler for optimal assembly. Specific as it says on the pb-assembly github page, "pb-assembly is the bioconda recipe encompassing all code and dependencies necessary to run" (https://github.com/PacificBiosciences/pb-assembly). The pb-assembly is stilled in the workflow and will be added on a later commit.

Since the pb-assembler seemed to be not working right now, it was continued with the hifiasm assembler and peregrine-2021 assembler. For that the hifiasm was installed via conda and run like, where -o specifies the output namings and -t the number of CPUs used.
```
hifiasm -o SRR135.asm -t 6 SRR13577846.fastq
```

The other as seemed better assembler used is Peregrine. Peregringe is a really fast and modern genome assembler, with easy to use commands. It was installed using bioconda in the version 0.4.13 and can be retrieved from https://github.com/cschin/peregrine-2021.
```
conda install -c bioconda peregrine-2021
```

Before the assembly with peregrine can bes started, the program requires to put the paths to the sequence read files (.../SRR13577846.fastq.gz) in a text file, e.g. called `seq.lst`. So that peregrine can find the read data. Also a output directory is created.

To start the assembly, the following command need to be run:
```
<<<<<<< HEAD


=======
```
>>>>>>> a17b163b1c0a425c0b98f133f2cb69e76332df23

## Quast quality assessment



