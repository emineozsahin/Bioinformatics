# Identif the structural changes of a genome from NGS data

Before approaching any software, having a bed file for annotations in hand is helpful. 

### Prepare a bed file from gtf format  
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

# CNVkit



```
for i in `ls *MarkUp.sorted.cns`;do basename="${i%MarkUp.sorted.cns}vcf.gz"; sbatch cnvkit2.run $i  $basename; done

for i in ../*.MarkUp.sorted.cns; do basename=${i%.MarkUp.sorted.cns}.call.cns; cnvkit.py call $i -m none --center biweight --drop-low-coverage --ploidy 4 -o $basename ;done

```

#Prepared a cnv.number file

```
for i in `less GENES_FOR_CNV_new_line.txt`; do for j in `less sample_order.txt`;do gene=$i; cordinates=`less $j|grep $gene|cut -f1,2,3,9`;echo $j $gene $cordinates;done ;done |sed 's/ /     /g' > copy_number_ploidy.txt
```

copy_number_ploidy.txt seems:

# Cnvnator





# [nQuire](https://github.com/clwgg/nQuire)
Identify the ploidy levels of the samples. To do so, I partitite the bed file to chromosome specific bed files. I used ploidy.run found in /home/emine85/nQuire/ploidy to produce *-bedcc.bin files for each chromosome. 

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
[a sample figure](https://github.com/emineozsahin/Bioinformatics/blob/main/Picture1.png)

histogram for each chromosome
```
for i in *_denoised.bin;do basename=${i%_denoised.bin}_histo.txt; ./../nQuire histo $i > $basename ;done
```


# ploidyNGS

