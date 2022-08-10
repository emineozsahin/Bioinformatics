# [SnpEff](https://pcingola.github.io/SnpEff/se_introduction/)

+ Build a database if it does not present in the snpEff database

Be careful where you build the database for snpEff. There could be two location:

#### First Location:  Anywhere you wish to build the database
 
1- Modify the config file by adding the genome

```
# get the config file 
wget https://github.com/pcingola/SnpEff/raw/master/config/snpEff.config

nano new_genome_info.txt

# Enter a genome information as follow
# S288C genome
S288C.genome : Saccharomyces cerevisae S288C2
S288C.chrmt.codonTable: Yeast_Mitochondrial

# CEN.PK genome
CEN.PK.genome : Saccharomyces cerevisae CEN.PK
CEN.PK.chrmt.codonTable: Yeast_Mitochondrial 

# Now add the genome info at the end of the config file which is a huge file.
cat new_genome_info.txt >> snpEffect.config
```

2- Make a directory at the path you want for genome.gff and sequences.fa 
```
cd /home/emineozsahin/
mkdir S288C
cd S288C
#download the fasta file and gff file (or other types as indicated in the documentation like gtf..) for the genome and modify the file names to the genome.gff and sequences.fa
# Threfore when you list the folder it will have two files. 
ls S288C
genes.gff  sequences.fa  

# Now modify the path which is found at the begining of the config file to the location of S288C folder which is genome data folder, for example:
nano snpEffect.config
data.dir = /home/emineozsahin/ 
```

3- It is time to build the database. Pay attantion to the -c flag as it is important to show the software which config file to be used. If it is not provided software will look at the second location which I explained below. 

```
snpEff build -c snpEffect.config -gff3 -v S288C

java -jar ~/projects/def-gvdmlab/Software/snpEff/snpEff.jar  build -c snpEffect.config -gff3 -v CEN.PK
```

This generates a file called snpEffectPredictor.bin in the data.dir which is ~/projects/def-gvdmlab/emine85 on graham

Now snpeff can be run to annotate the vcf file. Again -c flag is important and the snpEffect.config file is present on the path where I run the following code. 

```
snpEff S288C -c snpEffect.config -no-downstream -no-upstream -no-intergenic  -csvStats sample.csv  sample.vcf > sample.ann.vcf

java -jar ~/projects/def-gvdmlab/Software/snpEff/snpEff.jar CEN.PK -c snpEffect.config -no-downstream -no-upstream -no-intergenic  -csvStats Kolsch_CEN_PK_freebayes_snpeffect.csv  Kolsch_CEN_PK_freebayes.vcf  > Kolsch_CEN_PK_SnpEff_ann_freebayes.vcf

```

#### Second Location: The location where snpEff installed. 
For example, if the insatallation made with conda  ```conda install -c bioconda snpeff``` a similar paths with the following will be present. 

+ /home/emineozsahin/miniconda3/share/snpeff-5.0-0/./data/S288C/
+ /home/emineozsahin/miniconda3/share/snpeff-5.0-0/snpEff.config

There is no need to change the data.dir path in the config file. But, add the genome info at the end of the config file. Make the data/S288C folders and download the gff and fasta files and name them accordingly and build the database ```snpEff build -gff3 -v S288C```. This time there is no need for -c flag as software automatically look for the config file which is found in the software folder: /home/emineozsahin/miniconda3/share/snpeff-5.0-0/snpEff.config

Now you can run the software, but this time again there is no need for -c flag.

```
snpEff S288C -no-downstream -no-upstream -no-intergenic  -csvStats sample.csv  sample.vcf > sample.ann.vcf
```

#### report the variants

```
grep "YGR289C"  sample.ann.vcf  |awk -v FS=' ' -v OFS='\t' '{if(match($8,/ANN=([^;]+)/,m) split(m[0],a,"|")) print($1, $2, $4, $5, a[2], a[4],  a[10], a[11])}' > AGT1_CEN_PK_variants.txt

```

#### finding the gene names from gene IDs

```
variants=read.csv("EC1118ref_EC1118ann.csv")
genes=read.table("S288C.genes.txt")

for (i in 1:length(variants$gene_id)){index<-which(as.character(genes$V1) == as.character(variants$gene_id[i])) 
if (length(index) >0) {variants$gene_name[i]=as.character(genes$V2[index])} else {variants$gene_name[i]=""}
}

write.csv(variants, "EC1118ref_EC1118ann_genenames.csv")
```

# Bcftools to annotate

prepare a bed file from gff, sort, compress and index it

```
awk '{if(match($9,/gene=([^;]+)/,m)) print($1, $4, $5, substr(m[0],6), $7)}' CEN.PK113-7D_AEHG00000000.gff|sed 's/ / /g' > CEN_PK.bed

sort -k 1,1 -k2,2n  CEN_PK.bed > CEN_PK_sorted.bed

bgzip -c CEN_PK_sorted.bed > CEN_PK_sorted.bed.gz

tabix CEN_PK_sorted.bed.gz

```
use the bed file to annotate the regions 

```
bcftools annotate -a CEN_PK_sorted.bed.gz -c CHROM,FROM,TO,GENE -h <(echo '##INFO=<ID=GENE,Number=1,Type=String,Description="Gene name">')  Kolsch_CEN_PK_freebayes.vcf  >  Kolsch_CEN_PK_freebayes_ann.vcf 

```

