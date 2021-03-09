# [SnpEff](https://pcingola.github.io/SnpEff/se_introduction/)

+ Build a database if it does not present in the snpEff database

Be careful where you build the database for snpEff. There could be two location:

1-  Anywhere you wish to build the database

First modify the config file by adding the genome

```
nano new_genome_info.txt

# Enter a genome information as follow
# S288C2 genome
S288C2.genome : Saccharomyces cerevisae S288C2
S288C2.chrmt.codonTable: Yeast_Mitochondrial

# Now add the genome info at the end of the config file that could be downloaded from snpeff(https://github.com/pcingola/SnpEff/raw/master/config/snpEff.config) 
cat new_genome_info.txt >> snpEffect.config

```

Second modify the path to where you want to build the database 

```
# Change the path which is found at the begining of the config file to your desired location for example:
nano snpEffect.config
data.dir = /home/emine85/alignments/
```

Thirdly, make a directory at the path you indicated in the config file
```
cd /home/emine85/alignments/
mkdir S288C
cd S288C
#download the fasta file and gff file (or other types as indicated in the documentation like gtf..) for the genome and modify the file names to the genome.gff and sequences.fa
# Threfore when you list the folder it will have two files. 
ls S288C
genes.gff  sequences.fa  
```

Finally, it is time to build the database. pay attantion to the -c flag as it is important to have if you want to build database 
```
snpEff build -c snpEffect.config -gff3 -v S288C
```




1. The location where snpEff installed. For example if the insatallation made with conda  ```conda install -c bioconda snpeff```






