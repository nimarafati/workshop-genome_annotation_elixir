---
layout: default-overview
title: Prokaryote annotation
exercises: 50
questions:
  - How to run bacterial annotation?
  - What should I look in my assembly to go forward
objectives:
  - run prokka
---
# Prerequisites
For this exercise you need to be logged in to Uppmax.

Setup the folder structure:

```bash
cd ~/annotation/
conda activate bioinfo
export data=/home/data/data_annotation/
export bacterial_annotation_path=~/annotation/bacterial_annotation
mkdir -p $bacterial_annotation_path
export LC_ALL=C
```

In order to use the blast functionality from within Prokka (see below) we need to edit the file "/home/export_paths/export_path.sh" and comment out this line:

```
export PATH=$PATH:/usr/local/WUBlast
```
the line will then look like this :

```
#export PATH=$PATH:/usr/local/WUBlast
```

After editing the file, please log out of the terminal and then log back in.

Execute the command blastn -version and inspect the output. It should be something like:

```
tblastn: 2.9.0+
Package: blast 2.9.0, build Jun 13 2019
```


# Bacterial annotation using Prokka

We are providing you with 3 different bacterial genomes (one E coli, one chlamydia and one streptococcus).
You should make a group of 3 persons and each of you should do the exercises for 1 species.

```bash
cd $bacterial_annotation_path
ln -s $data/raw_computes/Escherichia_coli_genome.fa
ln -s $data/raw_computes/Chlamydia_trachomatis_genome.fa
ln -s $data/raw_computes/Streptococcus_genome.fa
```

Before running Prokka on genomes assemblies, it is a good step to start with checking the gene content of the assembly

## Checking genes in the assembly

[BUSCO](https://busco.ezlab.org/) provides measures for quantitative assessment of genome assembly, gene set, and transcriptome completeness. Genes that make up the BUSCO sets for each major lineage are selected from orthologous groups with genes present as single-copy orthologs in at least 90% of the species.

Run BUSCO on the 3 bacterial assemblies provided. We will select the lineage set of bacteria.

BUSCO is using augustus to run, as we have no administator rights on uppmax we need to source the $BUSCO_SETUP that will copy the augustus folder with the config files.

```
busco -i Chlamydia_trachomatis_genome.fa -o chlamydia_busco -m geno -c 8 -l /home/data/opt-byod/busco/lineages/bacteria_odb9
```
Look at the results of busco in short_summary_chlamydia_busco.txt

<br>:question:what do you think about this assembly? Is it a good one? can you see any potential problem with continuing the annotation?
<br>:question:how do you expect the annotation will be?

Do the same for the two other assemblies and answer those questions again (you just need to change the input assembly and the name of the output folder).

## Prokka

Prokka is a really easy tool to use for bacterial annotation.

You are going to use the same assemblies you used previously for Busco and link protein sequence of the three species

```
ln -s $data/raw_computes/uniprot-escherichia.fasta
ln -s $data/raw_computes/uniprot-chlamydia.fasta
ln -s $data/raw_computes/uniprot-streptococcus.fasta

prokka --help
```
The goal of the exercise is for you to learn how to use prokka and to annotate the 3 assemblies and then visualize them in IGV.  

Run prokka without any options and then with options of your choices (we encourage you to try at least the options --proteins and --rfam)    

<details>
<summary> :key: Click to see example on how to run prokka (you do not need to run all of them you can run prokka only with the output options and the two options --proteins and --rfam)</summary>  

<br>Running prokka with only the output option looks like this :  

  <br><code> conda activate prokka
  prokka Chlamydia_trachomatis_genome.fa --outdir prokka_Chlamydia
</code>  

<br>Running prokka with only --rfam looks like this :  

<br><code> prokka Chlamydia_trachomatis_genome.fa --rfam --outdir prokka_Chlamydia_rfam
</code>  

<br>Running prokka with only --proteins looks like this :  

<br><code> prokka Chlamydia_trachomatis_genome.fa --proteins uniprot-chlamydia.fasta --outdir prokka_Chlamydia_prot
</code>  

<br>Running prokka with --proteins and --rfam looks like this :  

<br><code> prokka Chlamydia_trachomatis_genome.fa --proteins uniprot-chlamydia.fasta --rfam --outdir prokka_Chlamydia_prot_rfam
</code>  

<br>You can try other options to see what you would need to modify in your own projects!

</details>


Look at the different results obtained :

:question:Do you see any differences with the different options and no options you used? (like for instance with or without --proteins)
<br>:question:Did you get the annotation you expected after the busco results?


Congratulations you have annotated bacterial genomes!

## Checking gene set completeness

BUSCO can also be used after the annotation to check if you found the genes you were expected or if something happened during the annotation and you lost genes. To do so you change the option "-m geno" by "-m prot" (you need to replace XXX by the proper name of the file).

```
conda activate busco
busco -i prokka_Chlamydia/PROKKA_XXXX.faa -o chlamydia_busco_prot -m prot -c 8 -l bacteria_odb10
```
You can do it for the two other genomes.

:question:Do you see a difference with the BUSCO of the genome?

<details>
<summary>:key: Click to see the solution .</summary>  
<br>The results of busco should be better for the annotation than for the genome.
This is due to the fact that prodigal does a better structural prediction during the annotation with Prokka than the prediction done during the run of Busco.

For eukaryotic annotation however BUSCO results for genes are slightly lower than the BUSCO results for the full genome, this is due to the fact that annotation method will always not predict everything.
It should not be too much of a difference either.

</details>
