# Multiple/pairwase Alignments 

Different multiple alignment algorithms are listed below. It is important to consider the size of the dataset and length of sequences when choosing which one to use.  Below is a brief overview of each algorithm. 

## [MAFFT](https://mafft.cbrc.jp/alignment/software/manual/manual.html) 

MAFFT is a progressive-iterative aligner that uses guide tree re-estimation for obtaining more accurate distance measures.  

Fast and accurate, especially suited for large datasets (e.g. up to 30,000 sequences, or fewer long sequences*).
Multi-threaded for faster alignment. 
Suitable for sequences with long, low homology N-terminal or C-terminal extensions
Suitable for sequences with long internal gaps (use L-ins-i algorithm)
* For long sequences, the algorithm is faster if sequences are closely related.  

## Efficient whole genome aligners for highly divergent sequences
### [LASTZ](http://www.bx.psu.edu/~rsharris/lastz/README.lastz-1.04.03.html#fmt_axt) 
### [Mauve](http://darlinglab.org/mauve/mauve.html) 

## [ClustalW](http://manpages.ubuntu.com/manpages/bionic/man1/clustalw.1.html)

ClustalW is a progressive aligner and should be restricted to small alignments.


## MUSCLE 

MUSCLE is  is a progressive aligner that features rapid sequence distance estimation using k-mer counting, progressive alignment using a profile function termed the log-expectation score, and refinement using tree-dependent restricted partitioning of the sequences.  The algorithm is described at http://nar.oxfordjournals.org/content/32/5/1792.full.pdf+html and a full manual is available at http://www.drive5.com/muscle/manual/index.html.  The defaults are optimized for best accuracy.  However, you can reduce the run-time on large alignments without too much reduction in accuracy by reducing the maximum number of iterations.  

Suitable for medium-large alignments up to 1000 sequences. 
Not suitable for sequences with low homology N-terminal and C-terminal extensions. 
Clustal Omega

## Clustal Omega 
It is a fast, accurate aligner suitable for alignments of any size. It uses mBed guide trees and pair HMM-based algorithm which improves sensitivity and alignment quality.  A full description of the algorithms used by Clustal Omega is available in the Molecular Systems Biology paper Fast, scalable generation of high-quality protein multiple sequence alignments using Clustal Omega. Latest additions to Clustal Omega are described in Clustal Omega for making accurate alignments of many protein sciences.  See also the Clustal Omega website for further details.  

Suitable for very large datasets of over 2000 sequences
Multi-threaded for faster alignment.
Suitable for sequences with long, low homology N-terminal or C-terminal extensions
Not suitable for alignment of sequences with large internal indels
