---
layout: default-overview
title: Comparing and evaluating annotations
exercises: 1h15
questions:
  - How can we compare two annotation?
  - How to merge different annotations?
objectives:
  - learn how to assess the quality of an annotation
  - complement a first annotation with a second one
---

## Prerequisites

Setup the folder structure:

```
cd
export data=/home/data/data_annotation/
export assessment=~/annotation/structural_annotation/assessment
export abinitio_augustus_path=~/annotation/structural_annotation/abinitio_augustus
mkdir -p $assessment
cd $assessment
```

# Introduction

In this exercise you will handle different annotation files:

• the pure abinitio one done with augustus (practical : Abinitio with augustus),

• the evidence-based done with MAKER

• the abinitio evidence-driven one done with MAKER.

• the official annotation from Ensembl

## overview

Evaluating an annotation can be done in different ways:

• Looking at the number of genes
It isn't so much a quality check as a measure of congruency - i.e. the resulting numbers don't tell you which of the two gene builds is more correct. Based on the known genomes from related species you can at least estimate if you have under or over-predicted gene content.

• Comparison against a reference
 This case is really rare in real life except if you work on model organisms. Here our reference annotation comes from EnsEMBL that have originally imported it from FlyBase. There is a huge difference between this annotation and ours. Obviously, a lot of manual labor and much more data has been put into the FlyBase annotation - and this highlights a common limitation of any computational pipeline. You will simply never reach the same level of quality and detail as seen in a manually curated reference annotation.

• Running busco on proteins obtained from the annotation  
 It provides a nice feeling about the quality of the annotation but is biased by the fact it focus only on well conserved genes between species during evolution. So, what about species specific genes ?

• In reference to the evidence alignments (AED score)  
 It is what Maker uses internally to select gene models. After synthesizing and annotating loci, the resulting model will be ranked against the filtered evidence alignments. The more congruent these two points of information are, the lower the 'annotation edit distance' (AED) will be. The AED score can be used to e.g. check an annotation for problematic models that may then be subjected to manual curation.

### Gene number

Prepare the folder and get the data needed.  

```
cd $assessment
mkdir complement
cd complement
ln -s ../../maker_evidence/maker_evidence/maker_annotation.gff maker_evidence.gff
ln -s ../../maker_abinitio/maker_abinitio/maker_annotation.gff maker_abinitio.gff
```

As already seen previously you can have a look at the statistics of an annotation with the **agat_sp_statistics.pl** script.  
As you will note, there are some differences - and of course, this is expected, since different approaches has been used to generate them.

```
conda activate agat
agat_sp_statistics.pl --gff maker_evidence.gff -o maker_evidence_stat.txt
agat_sp_statistics.pl --gff maker_abinitio.gff -o maker_abinitio_stat.txt
```
**COMMENT : to try if there is a real difference or remove**

Different methods can predict genes that are not in common (non-overlaping). To increase the sensitivity it could be important to create a non-redundant concatenated gene set. Let's do it for the MAKER annotations:
```
agat_sp_complement_annotations.pl --ref maker_abinitio.gff --add maker_evidence.gff -o maker_abinitio_cplt_by_evidence.gff
```
:question:How many genes have been added in this new maker_abinitio_cplt_by_evidence.gff annotation?

<details>
<summary>:key: Click to see the solution .</summary>
In that case only 1 gene and 1 RNA (it actually depends on your *ab-initio* training), in real life it would be more.
In our project we often complement evidence based annotation with *ab-initio* annotation.
</details>

### BUSCO

BUSCO is run before annotating to check if the assembly is good and therefore if the annotation will be good. It is also run after the structural annotation to then compare if we indeed find a number of genes corresponding of the first run of busco.

 Prepare the folder and get the data needed.  
```
cd $assessment
mkdir busco
cd busco
ln -s $data/genome/genome.fa
ln -s $assessment/complement/maker_abinitio_cplt_by_evidence.gff
 ```
 * BUSCO on the assembly

```
conda deactivate
conda activate busco
busco -i genome.fa -o busco_genome -m genome -c 8 -l arthropoda_odb10
```

The results is in run_busco_genome/short_summary_busco_genome.txt.

 * BUSCO on the annotation

First, let's extract the proteins form the annotation:
```
conda deactivate
conda activate agat
agat_sp_extract_sequences.pl -gff maker_abinitio_cplt_by_evidence.gff -f genome.fa -p -o maker_abinitio_cplt_by_evidence.fa
```

```
conda deactivate
conda activate busco
busco -i maker_abinitio_cplt_by_evidence.fa -o busco_annotation -m prot -c 8 -l arthropoda_odb10
```

The results is in run_busco_annotation/short_summary_busco_genome.txt.  

:question:if you compare the busco results what do you see?

<details>
<summary>:key: Click to see the solution .</summary>
First of all, keep in mind that we are using only 1 chromosome of a full assembly so the results are not good.
however, they are consistent with each other with around 0.5% of complete busco for both and we even to do slightly better with the annotation (still depend on you training of *ab-initio*).
</details>

### Comparison with the reference annotation

As with many tasks within bioinformatics, it is always a great idea to first look around for existing solutions. In the case of comparing annotations, there are in fact options already out there. One such example is genometools.

First create the working folder:
```
cd $assessment
mkdir compare_ref
cd compare_ref
```

Then, copy or sym-link the EnsEMBL reference annotation as well as yours:
```
ln -s $abinitio_augustus_path/augustus_drosophila.gff
ln -s ../complement/maker_abinitio_cplt_by_evidence.gff
ln -s $data/annotation/ensembl.genome.gff
```

Now we have to sort any GFF3-formatted annotation in a way that genometools accepts:
```
conda deactivate
conda activate genometools
gt gff3 -sort augustus_drosophila.gff > augustus_drosophila.sorted.gff
gt gff3 -sort maker_abinitio_cplt_by_evidence.gff > maker_abinitio_cplt_by_evidence.sorted.gff
gt gff3 -sort ensembl.genome.gff > ensembl.sorted.gff
```

With the sorted files, we can now perform a comparison two by two:
```
gt eval ensembl.sorted.gff augustus_drosophila.sorted.gff
gt eval ensembl.sorted.gff maker_abinitio_cplt_by_evidence.sorted.gff
```

This will create a long list of measures for all relevant sequence features with respect to both the 'sensitivity' and 'specificity' - as a measure of how close the annotation comes to a reference. As a reminder, 'specificity' measures the fraction of a reference overlapping a prediction whereas 'sensitivity' measures the fraction of a prediction overlapping a reference.

Note that the measures employed by genometools function in a all-or-nothing fashion. If the overlap is not 100%, it doesn't count (which is why you are unlikely to find gene-level congruencies between your gene builds and the reference annotation).  

:question:From the comparison of your annotations (the pure abinitio Augustus one and the one made with MAKER) to the Ensembl annotation, which one **seems** to be the most comprehensive to you ?

<details>
<summary>:key: Click here to see the expected output of ensembl vs augustus.</summary>
gene sensitivity (mRNA level): 100.00% (0/0) (missing genes: 0)<br>
gene specificity (mRNA level):   0.00% (0/60) (wrong genes: 60)<br>
gene sensitivity (CDS level): 100.00% (0/0) (missing genes: 0)<br>
gene specificity (CDS level):   0.00% (0/60) (wrong genes: 60)<br>
mRNA sensitivity (mRNA level):   0.00% (0/299) (missing mRNAs: 299)<br>
mRNA specificity (mRNA level): 100.00% (0/0) (wrong mRNAs: 0)<br>
mRNA sensitivity (CDS level):   0.00% (0/299) (missing mRNAs: 299)<br>
mRNA specificity (CDS level): 100.00% (0/0) (wrong mRNAs: 0)<br>
exon sensitivity (mRNA level, all):   7.31% (225/3078)<br>
exon specificity (mRNA level, all):  42.21% (225/533)<br>
exon sensitivity (mRNA level, single):   0.00% (0/11)<br>
exon specificity (mRNA level, single):   0.00% (0/3)<br>
exon sensitivity (mRNA level, initial):   0.00% (0/311)<br>
exon specificity (mRNA level, initial):   0.00% (0/57)<br>
exon sensitivity (mRNA level, internal):   9.20% (225/2445)<br>
exon specificity (mRNA level, internal):  54.09% (225/416)<br>
exon sensitivity (mRNA level, terminal):   0.00% (0/311)<br>
exon specificity (mRNA level, terminal):   0.00% (0/57)<br>
exon sensitivity (mRNA level, all, collapsed):  22.46% (225/1002)<br>
exon specificity (mRNA level, all, collapsed):  42.21% (225/533)<br>
exon sensitivity (mRNA level, single, collapsed):   0.00% (0/11)<br>
exon specificity (mRNA level, single, collapsed):   0.00% (0/3)<br>
exon sensitivity (mRNA level, initial, collapsed):   0.00% (0/184)<br>
exon specificity (mRNA level, initial, collapsed):   0.00% (0/57)<br>
exon sensitivity (mRNA level, internal, collapsed):  34.04% (225/661)<br>  
exon specificity (mRNA level, internal, collapsed):  54.09% (225/416)<br>
exon sensitivity (mRNA level, terminal, collapsed):   0.00% (0/146)<br>
exon specificity (mRNA level, terminal, collapsed):   0.00% (0/57)<br>
exon sensitivity (CDS level, all):   8.58% (229/2668)<br>
exon specificity (CDS level, all):  51.81% (229/442)<br>
exon sensitivity (CDS level, single):   0.00% (0/7)<br>
exon specificity (CDS level, single):   0.00% (0/4)<br>
exon sensitivity (CDS level, initial):   5.82% (17/292)<br>  
exon specificity (CDS level, initial):  30.36% (17/56)<br>
exon sensitivity (CDS level, internal):  10.21% (212/2077)<br>  
exon specificity (CDS level, internal):  65.03% (212/326)<br>
exon sensitivity (CDS level, terminal):   0.00% (0/292)<br>
exon specificity (CDS level, terminal):   0.00% (0/56)<br>
exon sensitivity (CDS level, all, collapsed):  30.37% (229/754)<br>
exon specificity (CDS level, all, collapsed):  51.81% (229/442)<br>
exon sensitivity (CDS level, single, collapsed):   0.00% (0/6)<br>
exon specificity (CDS level, single, collapsed):   0.00% (0/4)<br>
exon sensitivity (CDS level, initial, collapsed):  16.04% (17/106)<br>  
exon specificity (CDS level, initial, collapsed):  30.36% (17/56)<br>
exon sensitivity (CDS level, internal, collapsed):  40.00% (212/530)<br>
exon specificity (CDS level, internal, collapsed):  65.03% (212/326)<br>
exon sensitivity (CDS level, terminal, collapsed):   0.00% (0/112)<br>
exon specificity (CDS level, terminal, collapsed):   0.00% (0/56)<br>
nucleotide sensitivity (mRNA level):  46.95% (TP=191195/(TP=191195 + FN=216048))<br>
nucleotide specificity (mRNA level):  69.73% (TP=191195/(TP=191195 + FP=83008))<br>
nucleotide sensitivity (CDS level):  58.15% (TP=152785/(TP=152785 + FN=109956))<br>
nucleotide specificity (CDS level):  84.76% (TP=152785/(TP=152785 + FP=27479))<br>
</details>

<details>
<summary>:key: Click here to see the expected output of ensembl vs maker_abinitio_cplt_by_evidence.</summary>

gene sensitivity (mRNA level): 100.00% (0/0) (missing genes: 0)<br>
gene specificity (mRNA level):   0.00% (0/106) (wrong genes: 106)<br>
gene sensitivity (CDS level): 100.00% (0/0) (missing genes: 0)<br>
gene specificity (CDS level):   0.00% (0/106) (wrong genes: 106)<br>
mRNA sensitivity (mRNA level):   0.00% (0/299) (missing mRNAs: 2)<br>
mRNA specificity (mRNA level):   0.00% (0/106) (wrong mRNAs: 18)<br>
mRNA sensitivity (CDS level):   0.00% (0/299) (missing mRNAs: 2)<br>
mRNA specificity (CDS level):   0.00% (0/106) (wrong mRNAs: 18)<br>
exon sensitivity (mRNA level, all):  13.55% (417/3078)<br>
exon specificity (mRNA level, all):  61.41% (417/679)<br>
exon sensitivity (mRNA level, single):   0.00% (0/11)<br>
exon specificity (mRNA level, single):   0.00% (0/13)<br>
exon sensitivity (mRNA level, initial):   0.00% (0/311)<br>
exon specificity (mRNA level, initial):   0.00% (0/93)<br>
exon sensitivity (mRNA level, internal):  16.85% (412/2445)<br>
exon specificity (mRNA level, internal):  85.83% (412/480)<br>
exon sensitivity (mRNA level, terminal):   0.32% (1/311)<br>
exon specificity (mRNA level, terminal):   1.08% (1/93)<br>
exon sensitivity (mRNA level, all, collapsed):  41.62% (417/1002)<br>
exon specificity (mRNA level, all, collapsed):  61.41% (417/679)<br>
exon sensitivity (mRNA level, single, collapsed):   0.00% (0/11)<br>
exon specificity (mRNA level, single, collapsed):   0.00% (0/13)<br>
exon sensitivity (mRNA level, initial, collapsed):   0.00% (0/184)<br>
exon specificity (mRNA level, initial, collapsed):   0.00% (0/93)<br>
exon sensitivity (mRNA level, internal, collapsed):  62.33% (412/661)<br>
exon specificity (mRNA level, internal, collapsed):  85.83% (412/480)<br>
exon sensitivity (mRNA level, terminal, collapsed):   0.68% (1/146)<br>
exon specificity (mRNA level, terminal, collapsed):   1.08% (1/93)<br>
exon sensitivity (CDS level, all):  16.38% (437/2668)<br>
exon specificity (CDS level, all):  68.71% (437/636)<br>
exon sensitivity (CDS level, single):   0.00% (0/7)<br>
exon specificity (CDS level, single):   0.00% (0/14)<br>
exon sensitivity (CDS level, initial):  21.23% (62/292)<br>
exon specificity (CDS level, initial):  67.39% (62/92)<br>
exon sensitivity (CDS level, internal):  17.96% (373/2077)<br>
exon specificity (CDS level, internal):  85.16% (373/438)<br>
exon sensitivity (CDS level, terminal):   0.00% (0/292)<br>
exon specificity (CDS level, terminal):   0.00% (0/92)<br>
exon sensitivity (CDS level, all, collapsed):  57.96% (437/754)<br>
exon specificity (CDS level, all, collapsed):  68.71% (437/636)<br>
exon sensitivity (CDS level, single, collapsed):   0.00% (0/6)<br>
exon specificity (CDS level, single, collapsed):   0.00% (0/14)<br>
exon sensitivity (CDS level, initial, collapsed):  58.49% (62/106)<br>
exon specificity (CDS level, initial, collapsed):  67.39% (62/92)<br>
exon sensitivity (CDS level, internal, collapsed):  70.38% (373/530)<br>
exon specificity (CDS level, internal, collapsed):  85.16% (373/438)<br>
exon sensitivity (CDS level, terminal, collapsed):   0.00% (0/112)<br>
exon specificity (CDS level, terminal, collapsed):   0.00% (0/92)<br>
nucleotide sensitivity (mRNA level):  58.04% (TP=236347/(TP=236347 + FN=170896))<br>
nucleotide specificity (mRNA level):  95.88% (TP=236347/(TP=236347 + FP=10163))<br>
nucleotide sensitivity (CDS level):  81.94% (TP=215303/(TP=215303 + FN=47438))<br>
nucleotide specificity (CDS level):  94.55% (TP=215303/(TP=215303 + FP=12412))<br>
</details>  

### Filter MAKER annotation by AED score

A AED value of 0 means the whole gene model is supported by evidence while 1 means there is none. Let's try to select only models with good congruency with evidence lines, AED <0.3.

```
cd $assessment
mkdir filter
cd filter
ln -s ../complement/maker_abinitio_cplt_by_evidence.gff

conda deactivate
conda activate agat
agat_sp_filter_feature_by_attribute_value.pl --gff maker_abinitio_cplt_by_evidence.gff --value 0.3 -a _AED -t ">" -o codingGeneFeatures.filter.gff

```

:question:How many genes have passed your filter ? How many have been discarded ?

## Visualising annotations

**Note:** The following section overlaps with some of the exercises you have done earlier (comparing augustus predictions against the reference annotation).

In the previous tasks, we have looked at the overlap between different gene builds. While this gives us an indication of how similar two annotations are, **it doesn't really allow us to judge the overall quality and similarity of annotations**. Remember, sensitivity and specificity are 'all-or-nothing' - two annotations may be considered very different, but provide similar information, biologically. By that, we mean that two gene models don't need to be 100% identical in their coordinates to tell the scientist that a gene indeed exists in a given location and what it's product looks like.

We therefore need to visually inspect and compare the gene builds. This is a crucial step in any annotation project - gene build pipelines use a set of defined rules, but human pattern recognition is needed to spot potential systematic errors. For example, a pipeline like Maker will simply take all your input and try to synthesize it into an annotation, but it doesn't do too much checks on the data itself. What if you RNA-seq data is messier than you thought? What if your protein data set includes to many 'predicted' proteins that are in clear conflict with the other data?

There exist a number of 'annotation viewers' - IGV, Argo and Apollo, to name a few. A common choice for annotators is the web-based version of Apollo, WebApollo, mostly for its curation capabilities.

### Using WebApollo to view annotations
Transfer your maker annotation files to your computer using the scp command.  
Then, jump to [WebApollo](webapollo_usage) and upload your annotation track into the **dromel\_chr4** genome portal.
You can now compare your gene builds against this reference. Some questions to ask yourself:

:question:Do my gene builds recover all the genes found in the reference?  
:question:What sort of differences are most common?  
