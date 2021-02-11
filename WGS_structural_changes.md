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





# nQuire

```
./../nQuire create -b ../../scratch/alignments/sample11/Cali_Ale.MarkUp.sorted.bam -o S288C_$basename -r S288C_chrI.bed -y

for i in S288C_chr*;do temp=${i%.bed}; basename=`echo $temp | sed 's/S288C_/Cali_/'`; ./../nQuire create -b ../../scratch/alignments/sample11/Cali_Ale.MarkUp.sorted.bam -o $basename -r S288C_chrI.bed -y ;done

for i in S288C_chr*;do temp=${i%.bed}; basename=`echo $temp | sed 's/S288C_/Granvin_1_/‘`; ./../nQuire create -b ../../scratch/alignments/sample11/Granvin_1.MarkUp.sorted.bam -o $basename -r S288C_chrI.bed -y ;done
```
I used ploidy.run found in /home/emine85/nQuire/ploidy to produce *-bedcc.bin files for each chromosome. 

```
for i in *bedcc.bin;do echo $i ;./../nQuire view $i> ${i%-bedcc.bin}.txt;done 
```

Then following to retrieve the results.  
```
for i in *bedcc.bin;do ./../nQuire lrdmodel $i |grep -v "file";done > ploidy_chr.txt
```
plot the delta-log-likelihoods with any tool.

# ploidyNGS

