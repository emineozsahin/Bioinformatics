# Bioinformatics
My notes...
#### [Structural Analyses of genomes via NGS](https://github.com/emineozsahin/Bioinformatics/blob/main/WGS_structural_changes.md)
#### [Snakemake](https://github.com/emineozsahin/Bioinformatics/blob/main/snakemake.md)
#### [Alignment and snps extraction](https://github.com/emineozsahin/Bioinformatics/blob/main/Alignment.md)
#### [awk](https://github.com/emineozsahin/Bioinformatics/blob/main/awk.md) 
#### [snpEff](: Genetic variant annotation and functional effect prediction toolbox. 

# Tools
#### [SOAPdenovo2](https://github.com/aquaskyline/SOAPdenovo2)
####

# Resources and tutorials 
#### [PH525x series - Biomedical Data Science](http://genomicsclass.github.io/book/)
#### [HarvardX Biomedical Data Science Open Online Training](http://rafalab.github.io/pages/harvardx.html)
#### [Next-Gen Sequence Analysis Workshop (2019)](https://angus.readthedocs.io/en/2019/toc.html#)
#### [Analysis pipelines with Python](https://hpc-carpentry.github.io/hpc-python/)

# Programing languages
### [UNIX](https://tldp.org/LDP/Bash-Beginners-Guide/html/index.html)

+ [AWK](https://klashxx.github.io/awk-power-for-your-cmd) power for your command line.
+ [AWK GTF!](http://reasoniamhere.com/2013/09/16/awk-gtf-how-to-analyze-a-transcriptome-like-a-pro-part-1/) How to Analyze a Transcriptome Like a Pro... 

### [Python](https://www.w3schools.com/python/default.asp) 
I suggest to use [JupyterLab](https://jupyterlab.readthedocs.io/en/stable/index.html) instead of PyCharm.  [Jupyter Notebook configuration](http://stuartmumford.uk/blog/jupyter-notebook-and-conda.html) for the virtual environments. [ipython](https://ipython.readthedocs.io/en/stable/install/kernel_install.html)

[jupyter_lab on graham.computecanada](https://docs.computecanada.ca/wiki/Jupyter)

### [R](https://www.w3schools.com/r/default.asp)

### [C++](https://www.w3schools.com/cpp/default.asp)

# Virtual Environments and Containers
[Conda environments](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)

[Docker images and containers](https://docs.docker.com/engine/reference/builder/)

[A docker tutorial](https://github.com/docker/labs/blob/master/beginner/chapters/setup.md)

# Package management
There are several package managers such as pip, homebrew, apt-get, CRAN, CPAN and Bioconductor. [Conda](https://bioconda.github.io/tutorials/gcb2020.html#what-exactly-is-conda) is a combination of other package managers. [Bioconda](https://bioconda.github.io/index.html) is a channel for bioinformatics tools found in conda. [Biopython](https://biopython.org/wiki/Packages) can be installed via conda.        
#### [Biopython Tutorial and Cookbook](http://biopython.org/DIST/docs/tutorial/Tutorial.html)

Bioconda compose of a collection of bioinformatics [tools](https://anaconda.org/bioconda/repo?sort=_name&sort_order=asc&page=15) which can be installed easily.  


# High Performance Computing
### [AWS](https://aws.amazon.com) and [tibanna](https://tibanna.readthedocs.io/en/latest/)

### [Compute Canada](https://docs.computecanada.ca/wiki/Compute_Canada_Documentation)
Configure the local computer not to use password when connecting to graham.

Type ```ssh_keygen``` to the terminal. This will ask a name for the rsa key-pair. I am going to give it a name rsa_key. 

This will produce a rsa key file called rsa_key on the working directory. One can move this key-pair to the ~/.ssh folder. 

Now configure and resolve to access graham without password.

```
eval 'ssh-agent'
ssh-add ~/.ssh/rsa_key
```
Note that it is important to add the key file name after ssh-add command. Now we can log in cluster by typing 
```ssh  -i  ~/.ssh/rsa_key  username@graham.computecanada``` The first time of login it will ask passphrase which is the password but it won't ask a password for later logins. Adding an alias to the ~/.bash_profile file is also handy so that we can log in to clusters by just typing *graham*. 
```
nano ~/.bash_profile

#add the following line to the end of the file
alias graham="ssh -i ~/.ssh/rsa_key username@graham.computecanada.ca"
```

# Machine Learning
[TensorFlow](https://www.tensorflow.org)

# Reproducible workflow
[R Markdown](https://rmarkdown.rstudio.com) and [Snakemake](https://snakemake.readthedocs.io/en/stable/)

[Snakemake and containers](https://learning.cyverse.org/projects/Container-camp-2020/en/latest/breakout/workflows.html#setup) 

## [R_Markdown_Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
