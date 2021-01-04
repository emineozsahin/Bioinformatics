# Bioinformatics
Bioinformatics resources and tools...

# Resources
Two complementary resources are must-to-study resources for me. I strongly suggest them to anyone interested in learning computational biology. 
#### [PH525x series - Biomedical Data Science](http://genomicsclass.github.io/book/)
#### [HarvardX Biomedical Data Science Open Online Training](http://rafalab.github.io/pages/harvardx.html)

# Programing languages
### [UNIX](https://tldp.org/LDP/Bash-Beginners-Guide/html/index.html)
#### [Basics]()
#### [If statement and loops]()
#### [Tips and tricks]()
#### [AWK](https://klashxx.github.io/awk-power-for-your-cmd), power for your command line.
#### [AWK GTF!](http://reasoniamhere.com/2013/09/16/awk-gtf-how-to-analyze-a-transcriptome-like-a-pro-part-1/), How to Analyze a Transcriptome Like a Pro... 

### Python
#### []() 
I suggest to use [JupyterLab](https://jupyterlab.readthedocs.io/en/stable/index.html) instead of PyCharm.  

### R
[]()

### C++
[]()

# Virtual Environments

[Why you need python environments](https://www.freecodecamp.org/news/why-you-need-python-environments-and-how-to-manage-them-with-conda-85f155f4353c/)

# Package management
There are several package managers such as pip, homebrew, apt-get, CRAN, CPAN and Bioconductor. [Conda](https://bioconda.github.io/tutorials/gcb2020.html#what-exactly-is-conda) is a combination of other package managers. [Bioconda](https://bioconda.github.io/index.html) is a channel for bioinformatics tools found in conda. [Biopython](https://biopython.org/wiki/Packages) can be installed via conda.        
#### [Biopython Tutorial and Cookbook](http://biopython.org/DIST/docs/tutorial/Tutorial.html)

bioconda has a huge collection of bioinformatics [tools](https://anaconda.org/bioconda/repo?sort=_name&sort_order=asc&page=15). 

# Reproducible workflow
[R Markdown](https://rmarkdown.rstudio.com) and [Snakemake](https://snakemake.readthedocs.io/en/stable/)

### Snakemake
#### A detailed snakemake workflow
Samples are from [Snakemake](https://snakemake.readthedocs.io/en/stable/) tutorial. 

Start with a folder to test the snakemake and download the tutorial files.   

```
mkdir test
cd test
wget https://github.com/snakemake/snakemake-tutorial-data/archive/v5.4.5.tar.gz
tar -xf v5.4.5.tar.gz --strip 1 "*/data"
```

Miniconda is much smaller than Anaconda, and it is enough to install bioconda packages. Therefore, I suggest using miniconda to install software tools found in bioconda.
```

```

I prefer to create an environment rather than using my root environment. To do this I installed [mamba](https://mamba-framework.readthedocs.io/en/latest/what_mamba_is.html) as it is works faster than conda. Compare [conda and mamba](https://bioconda.github.io/tutorials/gcb2020.html)

```
conda install mamba -c conda-forge
```

To create a virtual environment, I used a [config file]() which says the mamba what packages to be installed to the environment.    

```
mamba env create -f envs/mapping.yaml 
```




# High Performance Computing
### AWS
### 

# Machine Learning
#### [TensorFlow](https://www.tensorflow.org)
