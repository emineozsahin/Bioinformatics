# Identify the structural changes of a genome from NGS data

Before approaching any software, having a bed file for annotations in hand is helpful. 

### Prepare a bed file from gtf 
get gtf file for *Saccharomyces cerevisiae* and unzip the file
```
wget ftp://ftp.ensembl.org/pub/release-102/gtf/saccharomyces_cerevisiae/Saccharomyces_cerevisiae.R64-1-1.102.gtf.gz
gunzip Saccharomyces_cerevisiae.R64-1-1.102.gtf.gz
```

convert the gtf file to an exon.txt file by using the script gtf2leafcutter.pl with the following code:
```
perl gtf2leafcutter.pl -o S288C Saccharomyces_cerevisiae.R64-1-1.102.gtf
```

Modify the exon.txt file via R and wrote the file as in a bed format. 
```
write.table(data, "S288C.bed", sep ="\t", quote=FALSE, row.names=FALSE, col.names=FALSE)
```

# [CNVkit](https://cnvkit.readthedocs.io/en/stable/)

I submitted a job but this could be a fast run.. 

```
#!/bin/sh
## This is to use cnvkit to produce .cns and .cnr files from bam files..

## Usage: sbatch cnvkit.run 

#SBATCH --account=def-*
#SBATCH --time=0-05:00:00 ## days-hours:minutes:seconds
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=11 # number of threads 
#SBATCH --mem=16000 # requested memory (in MB)
#SBATCH --job-name=cnvkit
#SBATCH --mail-user=<*@*>
#SBATCH --mail-type=END

cnvkit.py batch *MarkUp.sorted.bam -n \
    --targets S288C.bed  --annotate S288C_xenoRefFlat.txt \
    --fasta sequences.fna  \
    --output-reference S288C.cnn --output-dir cnvkit_results/ \
    --diagram --scatter -p 11
```
filter and change the chromosome names in cns and cnr files...

```
for i in *MarkUp.sorted*; do sh cnvkit_filter.sh $i;done
```

cnvkit_filter.sh:
```
#!/bin/bash

# files: MarkUp.sorted.cns and MarkUp.sorted.cnr

echo $1
basename=`echo $1 | sed 's/MarkUp.sorted/call/'`
echo $basename

# filter and adjust the cns and cnr files
cnvkit.py call $1 -m none --center biweight --drop-low-coverage --ploidy 4 -o  $basename

# resulted file name sample which is basename variable: Cali_Ale.call.cnr

file_output=`echo $basename | sed 's/call/call.chr/'`

# change the chr names
sed "s/chrIII/chr3/g" $basename |sed "s/chrII/chr2/g"|sed "s/chrIV/chr4/g"| \
        sed "s/chrIX/chr9/g"|sed "s/chrI/chr1/g"|sed "s/chrVIII/chr8/g"|sed "s/chrVII/chr7/g"|\
        sed "s/chrVI/chr6/g"|sed "s/chrV/chr5/g"|sed "s/chrXIII/chr13/g"|sed "s/chrXII/chr12/g"|\
        sed "s/chrXIV/chr14/g"|sed "s/chrXI/chr11/g"|sed "s/chrXVI/chr16/g"|sed "s/chrXV/chr15/g"|\
        sed "s/chrX/chr10/g" > $file_output

# clean up
if [[ -s $basename ]]
  then
  rm $1
  echo "removed" $1
else
  echo $basename "is empty. Something's fishy.."
fi


if [[ -s $file_output ]]
  then
  rm $basename
  echo "removed" $basename
else
  echo $file_output "is empty. Something's fishy.."
fi

```

or simply filter the cn{s,r} files, but if do not change the chr names they will not be in order on the plot:
```
for i in ../*.MarkUp.sorted.cns; do basename=${i%.MarkUp.sorted.cns}.call.cns; cnvkit.py call $i -m none --center biweight --drop-low-coverage --ploidy 4 -o $basename ;done

```
plot the copy ratios:

```
for i in  *filtered.cnr;do R_file=${i%cnr}cns; cnvkit.py scatter -s $i $R_file -o ${i%filtered.cnr}_scatter.pdf; done

```

Prepare a copy number file for selected genes. For this part I also added the vcf file to the analyses and then I retrieved the copy numbers. New cns files: 
```
for i in *MarkUp.sorted.cns; do vcf=${i%.MarkUp.sorted.cns}.vcf; sbatch cnvkit2.run  $i  $vcf;done
```
cnvkit2.run: 
```
#!/bin/sh
## This script

## Usage: sbatch cnvkit2.run  sample.MarkUp.sorted.cns  sample.vcf

#SBATCH --account=def-*
#SBATCH --time=0-01:00:00 ## days-hours:minutes:seconds
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1 # number of threads
#SBATCH --mem=16000 # requested memory (in MB)
#SBATCH --job-name=cnvkit
#SBATCH --mail-user=<*@*.ca>
#SBATCH --mail-type=END

echo $1
basename=${1%.MarkUp.sorted.cns}.call.cns

cnvkit.py call $1 -v ../freebayes_vcf/$2  --ploidy 4 -o $basename
```

Retrieve copy numbers
```
for i in `less GENES_FOR_CNV_new_line.txt`; do for j in `less sample_order.txt`;do gene=$i; cordinates=`less $j|grep -w $gene|cut -f1,2,3,9`;echo $j $gene $cordinates;done ;done |sed 's/ /     /g' > copy_number_ploidy.txt
```

# [nQuire](https://github.com/clwgg/nQuire)
Identify the ploidy levels of the samples. To do so, I separated the bed file into chromosome specific bed files. I used ploidy.run found in /home/emine85/nQuire/ploidy to produce -bedcc.bin files for each chromosome.

```
for i in S288C_chr*;do temp=${i%.bed}; basename=`echo $temp | sed 's/S288C_/Cali_/'`; ./../nQuire create -b /scratch/alignments/sample11/Cali_Ale.MarkUp.sorted.bam -o $basename -r S288C_chrI.bed -y ;done

```

Denoise the bin files: 
```
for i in *-bedcc.bin;do echo $basename; basename=${i%-bedcc.bin}_denoised ; ./../nQuire denoise $i -o $basename;done
```

Retrieve the results:
```
for i in *denoised.bin;do ./../nQuire lrdmodel $i |grep -v "file";done > ploidy_chr_denoised.txt
```

plot the delta-log-likelihoods via ggplot
[R Graphics Cookbook](https://r-graphics.org)

```
#Change the chromosome names from roman numerals 
sed "s/chrIII/chr3/g" ploidy_chr_denoised.txt |sed "s/chrII/chr2/g"|sed "s/chrIV/chr4/g"| sed "s/chrIX/chr9/g"|sed "s/chrI/chr1/g"|sed "s/chrVIII/chr8/g"|sed "s/chrVII/chr7/g"|sed "s/chrVI/chr6/g"|sed "s/chrV/chr5/g"|sed "s/chrXIII/chr13/g"|sed "s/chrXII/chr12/g"|sed "s/chrXIV/chr14/g"|sed "s/chrXI/chr11/g"|sed "s/chrXVI/chr16/g"|sed "s/chrXV/chr15/g"|sed "s/chrX/chr10/g" > ploidy_chr_denoised_ordered.txt

#start R
R

library(stringr)
library(reshape2)
library(ggplot2)

data=read.table("ploidy_chr_denoised_ordered.txt", sep="\t")
data$V1=gsub("_denoised.bin", "", data$V1)
data=cbind(str_split_fixed(data$V1, "_chr", 2), data)[,-3]
colnames(data)<- c("sample","Chromosomes","free","dip","tri","tet","diploid","triploid","tetraploid")
samples <- lapply(unique(data$sample), function(x) data[which(data$sample == x),])
names(samples) <- unique(data$sample)
samples <- lapply(samples, function(x)  x[,c("Chromosomes","diploid","triploid","tetraploid")])
samples <- lapply(samples, function(x) melt(x, id.vars="Chromosomes"))
for (i in names(samples)) {colnames(samples[[i]]) <- c("Chromosomes", "ploidy", "delta_Log_Likelihood")} 
samples <- lapply (samples, function(x) x[-which(x$Chromosomes == "mt"),])

for (i in 1:length(samples)) {samples[[i]]$Chromosomes=factor(samples[[i]]$Chromosomes, levels=c(1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16), ordered = TRUE)}

for (i in 1:length(samples)) {ggplot(samples[[i]], aes(Chromosomes, delta_Log_Likelihood, group=ploidy, colour=ploidy)) + geom_line() + geom_point(size = 1) +   scale_colour_brewer(palette = "Set1") + ggtitle(names(samples[i])) + ggsave(paste0(names(samples[i]), "_ploidy.pdf"), width = 8, height = 3, units = "in")}


```
[a sample ploidy figure](https://github.com/emineozsahin/Bioinformatics/blob/main/images/sample_Rplot.pdf)

histogram for each chromosome
```
for i in *_denoised.bin;do basename=${i%_denoised.bin}_histo.txt; ./../nQuire histo $i > $basename ;done
```
[a sample histo figure](https://github.com/emineozsahin/Bioinformatics/blob/main/images/sample_chrI_histo.txt)

# [ploidyNGS](https://github.com/diriano/ploidyNGS)

ploidyNGS is slow. For a 3Gb bam file it took 2 hours to finish the job. 

```
#!/bin/sh
## This script 
## Usage: sbatch 
#SBATCH --account=def-*
#SBATCH --time=0-5:00:00 ## days-hours:minutes:seconds
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16 # number of threads 
#SBATCH --mem=16000 # requested memory (in MB)
#SBATCH --job-name=ploidy
#SBATCH --mail-user=<*@*>
#SBATCH --mail-type=END

echo $1
basename=${1%.MarkUp.sorted.bam}_ploidyNGS

module load python/2

#for one sample:
#./ploidyNGS.py --guess_ploidy -b ../scratch/alignments/sample11/sample.MarkUp.sorted.bam -o sample_ploidyNGS

./ploidyNGS.py --guess_ploidy -b ../scratch/alignments/sample11/$1 -o $basename
```
result files: 
1-sample_ploidyNGS_depth100.ks-distance.PloidyNGS.tbl 
2-sample_ploidyNGS_depth100.tab
3-sample_ploidyNGS_depth100.tab.PloidyNGS.pdf
[a sample pdf](https://github.com/emineozsahin/Bioinformatics/blob/main/images/sample_ploidyNGS_depth100.tab.PloidyNGS.pdf)

# [cnvnator](https://github.com/abyzovlab/CNVnator)



