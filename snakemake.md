# SNAKEMAKE 
I took these notes while I was working on a snakemake pipeline which runs on AWS clusters for a variant analyses. 

## 1. Simple snakemake tutorial
[Slides](https://slides.com/johanneskoester/snakemake-tutorial)
#### I summurized a snakemake workflow which can be found [Snakemake](https://snakemake.readthedocs.io/en/stable/) website in detail. 

Start with a folder to test the snakemake and download the tutorial files.   

```
# test
mkdir test
cd test
wget https://github.com/snakemake/snakemake-tutorial-data/archive/v5.4.5.tar.gz
tar -xf v5.4.5.tar.gz --strip 1 "*/data"
```

Miniconda is much smaller than Anaconda, and it is enough to install bioconda packages. Therefore, I suggest using miniconda rather than Anaconda. 
Install [miniconda](https://docs.conda.io/en/latest/miniconda.html#macosx-installers)

miniconda can also be installed on terminal
```
curl https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh -o Miniconda3-latest-MacOSX-x86_64.sh
sh Miniconda3-latest-MacOSX-x86_64.sh
```

I prefer to create a virtual environment rather than using my root environment. [Why you need python environments](https://www.freecodecamp.org/news/why-you-need-python-environments-and-how-to-manage-them-with-conda-85f155f4353c/) To do this I installed [mamba](https://mamba-framework.readthedocs.io/en/latest/what_mamba_is.html) as it is works faster than conda. Compare [conda and mamba](https://bioconda.github.io/tutorials/gcb2020.html#introducing-mamba)

```
conda install mamba -c conda-forge
```

To create a virtual environment, I used a [config file](https://github.com/emineozsahin/Bioinformatics/blob/main/config/mapping.yaml) which says the mamba what packages to be installed to the environment.   

```
mamba env create -f envs/mapping.yaml 
```

In the config file the first line is to give a name to the environment which is *test* in this example. Therefore, activate the environment using the name *test*.
```
conda activate test
```
you can test your environment if snakemake and bwa are installed by typing `which snakemake` and `which bwa`

run the following code in the test folder which has the [__Snakemake__](https://github.com/emineozsahin/Bioinformatics/blob/main/test/Snakefile) file 

```
#This will create a mapped_reads folder with a bam file 
snakemake --use-conda --cores 1  mapped_reads/{A,B,C}.bam

#Deactivate the environment
conda deactivate

#remove the _test_ environment if it is not needed anymore
conda remove --name test --all
```

+ [expand](https://endrebak.gitbooks.io/the-snakemake-book/content/chapters/expand/expand.html)
+ workflow.snakefile
+ workflow.basedir
+ srcdir("filename")

## 2. Snakemake Wildcards tips

Some tips to use Snakemake wildcards:

```
DIRECTION = ["R1", "R2"]
SAMPLES = ["A", "B"]

rule all:
    input:
        expand("reads/{sample}_{direction}.fastq.gz", sample=SAMPLES, direction=DIRECTIONS)

rule copy_reads:
    input: 
        lambda wildcards: get_input_reads(wildcards, sample_annotation_df)
    output:
        'reads/{sample}_{direction}.fastq.gz'
    threads: 1
    group: "map_reads"
    resources:
	io=100,
        disk_mb=map_read_group_disk_mb
    shell:
        "cp {input} reads/"
```

When I ran the above Snakefile with ```snakemake -j 100``` the copy_reads rule is executed automatically. To do this *rule all* is essential and it has to have the outputs of the rule copy_reads. 

Now extend the Snakefile with second rule called fastqc.

```
rule all:
    input:
        expand("reads/{sample}_{direction}.fastq.gz", sample=SAMPLES, direction=DIRECTIONS)

rule copy_reads:
    input: 
        lambda wildcards: get_input_reads(wildcards, sample_annotation_df)
    output:
        'reads/{sample}_{direction}.fastq.gz'
    threads: 1
    group: "map_reads"
    resources:
	io=100,
        disk_mb=map_read_group_disk_mb
    shell:
        "cp {input} reads/"
       
       
rule fastqc:       
    input:
	expand("reads/{sample}_{direction}.fastq.gz", sample=SAMPLES, direction=DIRECTIONS)
    output:
        expand("reports/fastqc/{sample}_{direction}_fastqc.zip", sample=SAMPLES, direction=DIRECTIONS)
    group: 'map_reads'
    threads: 4
    resources:
	mem_mb=4000,
        disk_mb=map_read_group_disk_mb
    shell:
        "fastqc -q -o reports/fastqc -t {threads} {input}"

``` 

If I run this Snakefile via ```snakemake -j 100```, it won't give an error but it would not run the fastqc rule automatically. It would produce only the reads folder with the copied files. To run the fastqc rule ```snakemake -j 100 fastqc``` command has to be used. If I want to run fastqc rule automatically with just typing ```snakemake -j 100```, I have to add the outputs of fastqc to the *rule all*

```
rule all:
    input:
        expand("reads/{sample}_{direction}.fastq.gz", sample=SAMPLES, direction=DIRECTIONS),
        expand("reports/fastqc/{sample}_{direction}_fastqc.zip", sample=SAMPLES, direction=DIRECTIONS)

rule copy_reads:
    input: 
        lambda wildcards: get_input_reads(wildcards, sample_annotation_df)
    output:
        'reads/{sample}_{direction}.fastq.gz'
    threads: 1
    group: "map_reads"
    resources:
        io=100,
        disk_mb=map_read_group_disk_mb
    shell:
        "cp {input} reads/"
       
       
rule fastqc:       
    input:
        expand("reads/{sample}_{direction}.fastq.gz", sample=SAMPLES, direction=DIRECTIONS)
    output:
        expand("reports/fastqc/{sample}_{direction}_fastqc.zip", sample=SAMPLES, direction=DIRECTIONS)
    group: 'map_reads'
    threads: 4
    resources:
	mem_mb=4000,
        disk_mb=map_read_group_disk_mb
    shell:
        "fastqc -q -o reports/fastqc -t {threads} {input}"

``` 

Now when I type ```snakemake -j 100``` both rules will be executed and reports/fastqc and reads/ folders will be produced with their contents. 

There is another way to run rule fastqc automatically. I am going to add third rule which is called fastqc_report to the Snakefile. In this case the outputs of the rule fastqc is the inputs of the rule fastqc_report. I am going to add the *outputs of the fastqc_report* to the rule all and remove the *outputs of the rule fastqc* from the rule all. Because fastqc_report needs the outputs of the fastqc, fastqc will be executed without writing its outputs to the *rule all*.    

```
rule all:
    input:
        expand("reads/{sample}_{direction}.fastq.gz", sample=SAMPLES, direction=DIRECTIONS),
        "fastqc_scores_report.csv"

rule copy_reads:
    input: 
        lambda wildcards: get_input_reads(wildcards, sample_annotation_df)
    output:
        'reads/{sample}_{direction}.fastq.gz'
    threads: 1
    group: "map_reads"
    resources:
	      io=100,
        disk_mb=map_read_group_disk_mb
    shell:
        "cp {input} reads/"
       
       
rule fastqc:       
    input:
        expand("reads/{sample}_{direction}.fastq.gz", sample=SAMPLES, direction=DIRECTIONS)
    output:
        expand("reports/fastqc/{sample}_{direction}_fastqc.zip", sample=SAMPLES, direction=DIRECTIONS)
    group: 'map_reads'
    threads: 4
    resources:
        mem_mb=4000,
        disk_mb=map_read_group_disk_mb
    shell:
        "fastqc -q -o reports/fastqc -t {threads} {input}"
      
      
rule fastqc_report:
    input:
        expand("reports/fastqc/{sample}_{direction}_fastqc.zip", sample=SAMPLES, direction=DIRECTIONS) 
    output:
        "fastqc_scores_report.csv"
    shell: "python scripts/fastqc_report.py"

``` 

Here there is something one might think why I do not remove the outputs of rule copy_reads from rule all. I did not use wildcards in the input of the copy_reads but I identified them in the outputs of the copy_reads. Even though the get_input_reads function use the wildcards.sample and wildcards.direction, the meaning of the wildcards sample and direction are introduced to Snakemake in the outputs of the rule copy_reads. Snakemake does not accept the *__wildcards__* from outputs as an input of another rule. 

