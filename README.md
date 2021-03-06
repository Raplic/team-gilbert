
# Team-gilbert

This is HackBio repository for team-gilbert as a part of stage 1 of the internship

<h1 align="center">Hello 👋, We are Team-Gilbert 👋</h1>
<h3 align="center">The team of 10 scientists for HackBio-2020 Virtual Bioinformatics Internship </h3>



## Project

![HackBio](https://github.com/Team-Gilbert/team-gilbert/blob/hildred/stage1/hackbio.jpeg) <br>
Image Source: [@tbi_internship](https://twitter.com/tbi_internship)

The goal of this stage of the internship is to find biological data that can be analyzed with Linux based bioinformatics softwares, analyze it and provide visual representations of the results in a google drive folder.

The goal of our project is to make basical phylogenetic analysis with mycoplasma strains. Not only we provide insoghts in evolution of published *Mycoplasma hominis* strains, but also try to answer simple question - Do the choice of Multiple Sequence Alignment algorithm matters as long as your data is tidy and clean ?

<h1 align="center">Project: </h1>
<h1 align="center"> Comparative Genomics of Mycoplasma hominis </h1>

## Pipeline

![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/hildred/stage1/image.png)									



## What we want to answer
Do the chioce of Multiple Sequence Alignment algorithm matters if the we have good aligned core genomic data? We also provide insights in evolution *Mycoplasma hominis* strains
 

# Table of content
- [Team-gilbert](#team-gilbert)
  * [Project](#project)
  * [Pipeline](#pipeline)
  * [What we want to answer](#what-we-want-to-answer)
- [Quick start](#quick-start)
  * [Dependencies](#dependencies)
  * [Installation](#installation)
    + [Install Anaconda](#install-anaconda)
    + [Create Conda environments](#create-conda-environments)
    + [Installation of T-Coffee](#installation-of-t-coffee)
    + [Get the genbank_to_fasta script](#get-the-genbank-to-fasta-script)
    + [R packages installation](#r-packages-installation)
- [Usage](#usage)
  * [Step 1](#step-1)
  * [Step 2](#step-2)
  * [Step 2.5 (optional)](#step-25--optional-)
  * [Step 3](#step-3)
  * [Step 3.5 (optional)](#step-35--optional-)
  * [Step 4](#step-4)
  * [Step 5](#step-5)
  * [Step 6](#step-6)
- [Case study of Mycoplasma genomes](#case-study-of-mycoplasma-genomes)
  * [Goal](#goal)
  * [Flowchart of the project](#flowchart-of-the-project)
  * [Manual genome filtering and extraction](#manual-genome-filtering-and-extraction)
  * [Genome download and proteome extraction](#genome-download-and-proteome-extraction)
  * [Gather homologs of the curated set of proteins using BLAST against selected genomes](#gather-homologs-of-the-curated-set-of-proteins-using-blast-against-selected-genomes)
  * [Sequence extraction and concatenation](#sequence-extraction-and-concatenation)
  * [Phylogenetic reconstruction](#phylogenetic-reconstruction)
  * [Tree comparison](#tree-comparison)
- [TEAM MEMBERS](#team-members)



# Quick start

## Dependencies
- Conda (Anaconda/miniconda)
- T-Coffee package
- Genbank_to_fasta script from [Rocap lab](https://rocaplab.ocean.washington.edu/tools/genbank_to_fasta/)
- PareTree.jar script (provided in the repo)
- [R](https://www.r-project.org) + [Rstudio](https://rstudio.com/products/rstudio/download/) for tree comparison
- Linux enviroment with 
  - awk 
  - sed 
  - parallel
  - unzip
  - java 


## Installation

Majority of packaged are installed via conda command.
To satisfy linux dependencies please use standard package manager for your OS. 

### Install Anaconda
`wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh  && sh Miniconda3-latest-Linux-x86_64.sh` 

### Create Conda environments
`conda create -n gilbert-phylo muscle mafft dialign2 prank iqtree clustalo clustalw -y`
`conda create -n gilbert-prep blast prokka -y`
`conda create -n bio2 python=2.7 biopython -y`

### Installation of T-Coffee
`wget http://www.tcoffee.org/Packages/Stable/Latest/T-COFFEE_installer_Version_13.41.0.28bdc39_linux_x64.bin && chmod +x T-COFFEE_installer_Version_13.41.0.28bdc39_linux_x64.bin && ./T-COFFEE_installer_Version_13.41.0.28bdc39_linux_x64.bin`

### Get the genbank_to_fasta script
`wget https://rocaplab.ocean.washington.edu/files/genbank_to_fasta_v1.2.zip && unzip genbank_to_fasta_v1.2.zip `

### R packages installation
In order to visualize tree comparison we are relying on tanglegram library within R language. To run subsequent analysis properly you need several libraries to be installed
```R
install.packages("dendextend", "ape", "phylogram", "dplyr")
```


# Usage

**Note: This example use Mycoplasma genomes, but you can use any you want as long as you generated csv file from Genome tab in NCBI**

## Step 1 
Get the csv file of genomes of interest. We use the basic NCBI generated file from Genome tab.

## Step 2 
Run the following script from the repo folder:
```bash
mkdir genomes && cd genomes && sh genome_download.sh
```
**Note: In the script we are using `Mycoplasma_genomes.csv` file. So if you have different filename please change the script or the filename**

## Step 2.5 (optional)
Rename the gbff files to your purposes 

## Step 3
Copy the gbff files to the script folder, or point ti theirs location in `ls` command.
Run the genbank_to_fasta script:
``` bash
conda activate bio2 ;
cd genbank_to_fasta_v1.2;
ls ../genomes/*.gbff | parallel ./genbank_to_fasta.py -i {} -q 'locus_tag'
```
**Note: We used locus tag as only qualifies for fasta headers as it was constantly present in out dataset. Your choice of qualifier might depend on your dataset. We propose to use simple 1 feature qualifier, so that grep could find your genes from fasta later**

## Step 3.5 (optional)
If some of your genomes miss `CDS` feature, you need to annotate them. We propose to [use prokka](https://github.com/tseemann/prokka). Please consult with `-h` flag, but the the most minimal run looks like:
```bash
conda activate gilbert-prep; #activates environment with prokka installed
prokka your_sequence.fna
```
**You need to extract fna files from folders in order to run prokka**

## Step 4
Run blast on your subset of extracted protein sequences from genomes:
```bash
conda activate gilbert-prep
sh blast_pipeline.sh
```
**Note: the final file is named mycoplasma fasta. By default all the temporary files are removed. Comment the corresponding lines to keep them**

## Step 5
Actual running of the MSA algorithms with subsequent run of iqtree software on them. The every MSA algorithm should be run in the folder.

```bash
#Activate conda enviroment
conda activate gilbert-phylo
#Run MSA using Clustal Omega
mkdir clustalo && cd clustalo
clustalo --in=input.fasta && iqtree -s alignment.fasta
# Run MSA using ClustalW
mkdir clustalw && cd clustalw
clustalw [-infile] outfile.fasta && iqtree -s alignment.fasta
# Run MSA using Mafft
mkdir mafft && cd mafft
mafft in > out && iqtree -s alignment.fasta
# Run MSA using Muscle
mkdir muscle && cd muscle 
muscle -in input.fasta -out output.fasta && iqtree -s alignment.fasta
# Run MSA using PRANK
mkdir prank && cd prank
prank -d=input_file -t=tree_file -o=output_file && iqtree -s alignment.fasta
# Run MSA using T-coffee
mkdir t-coffee-simple && cd t-coffee-simple
t_coffee in-file.fasta && iqtree -s alignment.fasta;
cd ..
mkdir t-coffee-meta && cd t-coffee-meta
t_coffee in-file.fasta -mode mcoffee in-file.fasta
```
## Step 6
After runnnig the phylogenetic software the last step is pairwise comparison of computed trees. This is done via "tanglegram" algorithm.
1. But the first step is to delete branch lenghts from treeefiles. We don't really need step form the topology comparison step, but if they are in the file, two of the best approaches for comparison (step1wise and ste2wise) are unavailable. Therefore we decided to delete  branch lenghts.
This is done via [PareTree script](http://emmahodcroft.com/PareTree.html).
`ls *.treefile | parallel java -jar PareTree1.0.2.jar -f {} -topo`

2. Second step is R script running for tree comparison. There is no automation here, so you need to open the script in Rstudio and run every comparison manually. The only thing you need to alter is filenames in `read.tree` function and names of the trees in `tanglegram` function.

After run of an algorithm , you can export generated plots from Rstudio.


# Case study of Mycoplasma genomes

## Goal
We used the abive mentioned pipeline to gains insights about evolution of *Mycoplasma hominis* strains. And, hence, the genomes of these bacteria the small, we tried to identify core gene set for successful phyllogenetic tree reconstruction, using those, identified by Safa Boujemaa et al. (in this [article](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6173709/)). Not only we tried to refine the set of genes, but also see if the choice of Multiple Sequence Alignment algorithm at the first stage of phyllogeny recostruction really matters

- Pros of our approach is a usage of concatenated sequences of core genes from Mycoplasma strains. Therefore we have good phylogenetic signal to clearly distinguish between strains.
- The 16S rRNA approach is suitable only for species indentification (not strains-specific phylogeny)

## Flowchart of the project

![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/Stage_1_Pipeline.png)

## Manual genome filtering and extraction
We used "Browse by organism" in NCBI Genomes.
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/Genome_tab_1.png)

After searching for Mycoplasma hominis we ended with 37 genomes with different assembly quality:
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/Genome_tab_2.png)

After aplying filers, only 20 of the genomes have good enough assembly quality to be analysed:
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/Genome_tab_3.png)

After we downloaded csv file with 20 genomes of *Mycoplasma hominis* strains
## Genome download and proteome extraction
For downloading genomes we used `rsync` [method](https://www.ncbi.nlm.nih.gov/genome/doc/ftpfaq/)
1. Therefore we extracted GenBank FTP server links to the separate file, replaced `ftp:` with `rsync:` and iterated through each line with while loop. This method is not the most efficient one (parallel command works better). This was done by `genome_download.sh` script.
2. When all gbff files were extracted (we used gbff files because of their presence in every downloaded genome folder), we converted them to fasta, using Rocap Lab script. 
3. 4 generates fasta files had 0 bytes size, therefore we extracted `fna` sequences from corresponding genome folder and annotated them with prokka. We haven't used much parameters, because all we needed was only CDS identification.
4. 4 remaining gbk files were coverted to fasta using genbank_to_fasta script. This was done to extract only 'locus_tag' feature
5. All files were renamed using corresponding strain names for easy usage

## Gather homologs of the curated set of proteins using BLAST against selected genomes
Proposed by Safa Boujemaa et al. set of 10 genes is `gyrB, tuf, ftsY, uvrA, gap, p120', vaa, lmp1, lmp3, p60`. At first we indified those genes in all strains using blast. Then we run MSA (mafft) to see if the is with good quality. As we are using artificial concatenated sequences, introduced gaps are not having any biological sense. This alignment of artificial sequences must as clean as possible, with no big gaps introfuced,
The MSA with proposed genes looked not so good:
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/alignment_bad.png)
We can see that some genes introducing long gaps is several strains. As the the strain data assembly level was 'complete' or 'chromosome' we hypothesised that gene set was incorrect. After we deleted buch of genes from initial set (lmp1, lmp3, vaa as they were too variable) we eneded up with comperhensive set of 7 genes, whose concatenated sequences MSA looks much better:
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/alignment_good.png)

After the query gene set was refined, we run blast, to find those genes in each and every strain genome.
For this purpose we run :
```bash
conda activate gilbert-prep
sh blast_pipeline.sh
```
This was done in the folder, that is containing fasta files, as extracted CDS from every genome. In this folder query folder was available with query set of proteins (`Phylogeny_proteins.fasta`)

**Note: For query gene set refining we actually already run blast. So this step was done multiple times to find the most conserved gene set**
Blast output threshold was:
- E-value < 1e-10
- Subject coverage per HSP > 50%
- Best subject hit only

**Note: we used Best Hit strategy, not the most popular and more reliable Reciprocal Blast Hit one. This was done due to homolog finding problem is different strains, not species or organism from same genus. However we haven't used the best strategy, close evolutionaty relationship of the analyzed organisms allows us to conclude that BH and RBH results would be not different**

## Sequence extraction and concatenation
This step is part if a `blast_pipeline.sh` script. Steps for found fasta sequeces extraction and concatenation are the following:
1. Extract the 'Subject' column from the tabular blast output. Write it to the separate file
2. Create separate files for every proteome, where protein sequences would be in one line after fasta header
3. Search for every subject header in corresponding linerized fasta file and if found write it and 1line after (corresponding sequence) to separate file.
4. Concatenate those sequences in one big sequence for every file. Put clean sequence name as a header (filename in this case)
5. Add newline to the end of every file and concatenate themm in one multifasta file.

## Phylogenetic reconstruction
For phylogenetic reconstruction we used the output multifasta file, generated in the previous step. All the algorithms for MSA were installed using conda and the overall ran is similar to the 'Usage':
```bash
#Activate conda enviroment
conda activate gilbert-phylo
#Run MSA using Clustal Omega
mkdir clustalo && cd clustalo
clustalo --in=input.fasta && iqtree -s alignment.fasta
# Run MSA using ClustalW
mkdir clustalw && cd clustalw
clustalw [-infile] outfile.fasta && iqtree -s alignment.fasta
# Run MSA using Mafft
mkdir mafft && cd mafft
mafft in > out && iqtree -s alignment.fasta
# Run MSA using Muscle
mkdir muscle && cd muscle 
muscle -in input.fasta -out output.fasta && iqtree -s alignment.fasta
# Run MSA using PRANK
mkdir prank && cd prank
prank -d=input_file -t=tree_file -o=output_file && iqtree -s alignment.fasta
# Run MSA using T-coffee
mkdir t-coffee-simple && cd t-coffee-simple
t_coffee in-file.fasta && iqtree -s alignment.fasta;
cd ..
mkdir t-coffee-meta && cd t-coffee-meta
t_coffee in-file.fasta -mode mcoffee in-file.fasta
```
Not only the MSA algorithm is a variable that is changing in our analysis, but we also lesy the 'Modelfinder', which works under iqtree software, to fit the best model to the given alignment. This was done on purpose, as investigator usually wants best model for the given data. Therefore is more reflects research process. 

The iqtree parameters were default, as well as for all MSA algorithms. To look deep in mycoplasma evolution we chose MSA generated by mafft:

![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/tree_lenght.png)

As we can see from the tree above, the SP2565 and 4788 strains have the most brach lenghts. This can be due to instroduced gaps. It seems that those strains hols truncated versions of several proteins. To solve this issue more conservative set of genes can be found. We for out our our purposes this dataset is perfect, as it have some noise, shich can be interpretated	differently with different algorithms

If we ignore branch lenghts we would end up with wmth like this:
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/tree.png)

From the tree we can see there is such clusters as Safa Boujemaa et al. was describing. Maybe they were introduced by non-conservative genes, included in the analysis. Hovewer small cluster with strains 475, 4235 and FBG is visible. We can also conclude that strain pairs A136 + SS25, 8958 + SP10291 and SP3615 + SS10 are closely related.


As the branch length aclually indicates the number of mutations, that were introduced to the analyzed sequence, we can conclude that strain 4788 is the most "evolutionary modified" from the other *Mycoplasma hominis* strains 


Subsequent `*.treefile` files were extracted from folders and compared with tanglegram algorithm.

## Tree comparison

The folder 'data/6.tree_comparisons' holding all the pirwise comparisons of the trees
1. Clustalw is actually the oldest algorithm. It's results it different from every other:
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/clustaw-mafft.png)
2. Clustal Omega is a rethinked ClustalW. However it's results aren't much more similar compared to other algorithms:
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/clustao-mafft.png)
However the ovarall result is better
3. Muscle is the most widely used algorithm for middle set of genes to align:
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/muscle-mafft.png)
Interesting thing about the trees geenrates by this algorithm is thas they are 'upside-down'. As the other programs think that the most "evolutionary modified" organism is the opposite one.
However great similar results with `PRANK`
4. Mafft is the most widely used algoritm for large datasets. It is very fast. 
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/mafft-mcoffee.png)
Results of the mafft are still different from othe algorithms. Worth noting that the mafft-tcoffee comparison is perfectly similar!
5. T-Coffee claims to score the alignment, but it's gives not the best alignments. In our case it's results were not that far fom muscle and mafft/
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/muscle-tcoffee.png)
6. M-Coffee claims to combine the results of many MSA algoritms. It's result is different from any other algoritm/
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/mcoffee-prank.png)
7. PRANK is an algorithm that 'knows phylogeny'. It's results are far different from any other algoritm
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/prank-dialign.png)
8. The we can say about dialign. Very different results
![Workflow](https://github.com/Team-Gilbert/team-gilbert/blob/master/images/clustao-dialign.png)


**So, the choice of the MSA algorithm matters. We have remain a little bit of noise in our data and those algorithms interpreted this noise differently. So data cleaning is the most important step in phylogenetic reconstruction, as if there a little bit of noise it can be interpreted differently. Even popular MSA tools in out dataset gave similar, yet different results.**

# TEAM MEMBERS
- @Ayodeji
- @grape
- @Haniel
- @Highzhikk
- @Hildred
- @KHATEN
- @Oluwatomisin
- @pavlo
- @Rajesh
- @Semilogo