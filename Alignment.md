# Pairwise and Multiple Alignment 

## [BLAST+](https://www.ncbi.nlm.nih.gov/books/NBK279690/)

First needs to prepare a database, then align the query with database. 

```
module load  StdEnv/2020  gcc/9.3.0 blast+/2.11.0

makeblastdb -in sequences.fa -title S288C -dbtype nucl -num_alignments 10000 -out S288C_blast.fa

blastn -db S288C_blast.fa -query sample.fa > sample_blast_S288C.ref

```

If query sequence has different headers in a fasta file, split the sequences. 

```
module load StdEnv/2018 nixpkgs/16.09  intel/2018.3 kentutils/20180716

faSplit sequence genome.fasta 16 genome

```

Different multiple alignment algorithms are listed below. It is important to consider the size of the dataset and length of sequences when choosing which one to use.  Below is a brief overview of each algorithm. 

## [MAFFT](https://mafft.cbrc.jp/alignment/software/manual/manual.html) 

MAFFT is a progressive-iterative aligner that uses guide tree re-estimation for obtaining more accurate distance measures.  

+ Fast and accurate, especially suited for large datasets (e.g. up to 30,000 sequences, or fewer long sequences*).
+ Multi-threaded for faster alignment. 
+ Suitable for sequences with long, low homology N-terminal or C-terminal extensions
+ Suitable for sequences with long internal gaps (use L-ins-i algorithm)
+ Output is suitable for extracting the snps using the software snp-sites.

\*The algorithm is faster if sequences are closely related for long sequences. I used it to align the yeast chromosomes and it work well and fast. 

An example code:
147 SARS-CoV-2 whole genome sequences (29000bp)can be aligned under a minute with MAFFT aligner.  

```
module load StdEnv/2018 nixpkgs/16.09  gcc/7.3.0 mafft/7.397

mafft --auto --nuc gisaid_hcov-19_2020_04_26_21.fasta > covid_gisaid.aln
```

## Efficient whole genome aligners for highly divergent sequences
1. [LASTZ](http://www.bx.psu.edu/~rsharris/lastz/README.lastz-1.04.03.html#fmt_axt) 

A sample code for lastz. This software has many output options.     

```
module load lastz

lastz EC1118_chrII.fa  S288C_chrII.fa --format=maf > EC1118_vs_S288C_chrII.maf
 ```

2. [Mauve](http://darlinglab.org/mauve/mauve.html) 

gtf files can be aligned with mauve. I aligned the Adenovirus genomes. 

```

```

## Clustal Omega 
It is a fast, accurate aligner suitable for alignments of any size. It uses mBed guide trees and pair HMM-based algorithm which improves sensitivity and alignment quality [Sievers, F., et. al, 2011](https://www.scienceopen.com/document_file/75d83110-c338-403b-97fe-813d5e6b41f6/PubMedCentral/75d83110-c338-403b-97fe-813d5e6b41f6.pdf). 

+ Suitable for very large datasets of over 2000 sequences
+ Multi-threaded for faster alignment.
+ Suitable for sequences with long, low homology N-terminal or C-terminal extensions
+ Not suitable for alignment of sequences with large internal indels
+ Output is suitable for extracting the snps using the software snp-sites

Alignment of 147 SARS-CoV-2 whole genome sequences (29000bp) takes a very long time, so it is better to use queueing system.  

```
module load nixpkgs/16.09  gcc/5.4.0 clustal-omega/1.2.4

clustalo -i gisaid_hcov-19_2020_04_26_21.fasta -o clustalomega_out.fa -v -t DNA

```

## [ClustalW](http://manpages.ubuntu.com/manpages/bionic/man1/clustalw.1.html)
ClustalW is a progressive aligner and should be restricted to small alignments.

```
```

## [MUSCLE](http://www.drive5.com/muscle/manual/index.html) 
MUSCLE is a progressive aligner that features rapid sequence distance estimation using k-mer counting, progressive alignment using a profile function termed the log-expectation score, and refinement using tree-dependent restricted partitioning of the sequences [Drive & Valley, 2004](http://nar.oxfordjournals.org/content/32/5/1792.full.pdf+html). The defaults are optimized for best accuracy.  However, one can reduce the run-time on large alignments without too much reduction in accuracy by reducing the maximum number of iterations.  

+ Suitable for medium-large alignments up to 1000 sequences. 
+ Not suitable for sequences with low homology N-terminal and C-terminal extensions. 

```
```
## Visualize the alignment
[Jalview](http://www.jalview.org/getdown/release/) is an efficient tool to see the alignment to make sure the alignment is good.   

## [snp-sites](https://github.com/sanger-pathogens/snp-sites) 
To extract the variants from an alignment file.

```
# installation
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p $HOME/miniconda

conda config --add channels conda-forge
conda config --add channels defaults
conda config --add channels r
conda config --add channels bioconda
conda install snp-sites

# run the snp-sites
snp-sites -v -o gisaid_covid19.vcf gisaid_covid19.aln

```
after snp-sites the vcf files cannot be annotated by snpEff unless chromosome names format would be changed from integer to the style chr[I-X].
Change chromosome names of all vcf files found at the path

```
# for one file 
head sample.vcf -n4 > sample_out.vcf; tail sample.vcf -n+5|awk '$0=gensub(/1/, "chrI", $1)' >> sample_out.vcf

# for all the vcf found in the path 
for i in EC1118_chr*_EC1118_*.vcf;do chr=`echo $i |sed 's/_/^V/g'|cut -f2`; outfile=`echo $i | sed 's/.vcf/_chr/g'`;echo $chr $outfile;head $i -n4 > $outfile;tail $i -n+5 | awk -v chr="$chr" '$0=gensub(/1/, chr, $1)' >> $outfile; done
```

Files are ready to annotate 

```
for i in *chr; do csv=`echo $i|sed 's/chr/csv/g'`; outname=`echo $i|sed 's/_chr/_ann.vcf/g'`; echo $csv, echo $outname; snpEff S288C -c snpEffect.config -no-downstream -no-upstream -no-intergenic -csvStats $csv $i > $outname ; done

```









