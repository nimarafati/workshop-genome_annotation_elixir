---
layout: default-overview
title: Manual curation
exercises: 60
questions:
  - How do I load data into a genome browser?
  - How do we modify gene models?
  - How to interpret the different tracks?
objectives:
  - Be confident to navigate into a genome browser.
  - Learn how to modify gene models.
  - Learn how to interpret different tracks to build/correct a gene model.
---

## Overview

It is easy to understand that automated gene build pipelines will never reach 100% accuracy in their reconstruction. This is due to a number of factors, including ambiguous information from competing input data, inherent uncertainties of ab-initio predictions as well as simplified decision processes when synthesising all available information into a transcript structure. It is therefore always important to manually inspect a gene build - and in basically all cases manual curation is highly recommended.

Manual curation is a common step in any genome project, often referred to as a jamboree. All researchers involved in the project will meet - virtually or physically - and together inspect the gene build(s) to correct remaining issues prior to publication or downstream analyses. Here we will learn about manual curation tools and best practices, which you can then employ in your own annotation project.
## Meet: WebApollo

You have already encountered WebApollo in the previous exercise on gene building. There, you used its visualisation capabilities to look at several gene builds and compared them against the evidence alignments. However, what do you do if you find problems with your annotation? Basically, there are two options:

- The problems seem systematic and related to issues with the input data or settings.

In this case the best is to investigate and eliminate the issue(s) from the raw data and re-run the pipeline. Examples would be poorly assembled RNA-seq data or incompletely or badly sampled protein data. Another issue may be severe problems with the genome assembly. This of course is outside of your annotation task - and a discussion with the assembly team may be necessary.

- The problem is sporadic and looks otherwise non-systematic and complex

Complex, non-systematic errors are harder to rectify by just rerunning the pipeline. The goal of the computational gene build should be to generate a solid basis on which to build future analyses. An error rate of 20% is well within the expected margins and it is important to remember that a computational prediction will always be of lesser quality than a manually curated annotation. A sensible suggestion is to under-shoot rather than over-shoot. In other words, it is often better to be a little more conservative rather than to include as much information as possible. This is controlled by e.g. the way you have compiled your input data and settings within maker.

### Using WebApollo to curate gene models

Manual curation is an integral part of any annotation project. It reveals issues that exist in the gene build and can be used to add further detail - like references to external data sources, or isoforms etc.

The aim of manual curation is to compare a gene model against existing evidence from sources such as ab-initio predictions, protein alignments, RNA-seq as well as related species and fix those parts that are in clear conflict with the evidence. During the course, we will present a few basic features of WebApollo - but there is also a fairly comprehensive handbook available here: [http://icebox.lbl.gov/webapollo/docs/webapollo_user_guide.pdf](http://genomearchitect.github.io/users-guide/)

## Jamboree

For this exercise, we have set up a specific [Webapollo](http://annotation-prod.scilifelab.se:8080/NBIS_course) instance of a drosophila melanogaster annotation of the chromosome 4. It is called **drosophila\_melanogaster\_chr4\_jamboree**.  

The tracks available are:  

- Augustus_drosophila : a pure ab initio annotation using Augustus with the drosophila model.
- Maker\_evidence : A maker annotation using Evidence-based approach. This track is loaded as user track on top.
- Maker\_abinitio : A maker annotation using Ab initio evidence-drived approach.  
- Proteins : track of reviewed proteins aligned by Maker.
- tophat_larva4 : RNAseq data (bam file) aligned to the genome by tophat.  
- Cufflinks_larva4 : A cufflinks transcript assembly aligned by MAKER.
- Stringtie_ERR305399 : A stringtie transcript assembly aligned by MAKER.
- EST_from_NCBI : The ESTs aligned by maker during the annotation process.

A genomic region of the chrosmosome is assigned to each of you. Your aim is to manualy annotate your assigned part using all the information available in the different tracks. Genomic region has been assigned without any biological consideration. So, if genes straddle two regions don't stop you at the end of yours :).  

NOTES: Isoforms are allowed. Start each gene annotation by dragging-and-dropping the gene model that you think be the best.

1 :    	            1 - 168 982
<br/>2	:        168 983 - 337 965
<br/>3	:              337 966 - 506 948
<br/>4	:            506 949 - 675 931
<br/>5	:              675 932 - 844 914
<br/>6 :             844 915 - 1 013 897
<br/>7 :                1 013 898 - 1 182 880
<br/>8	:            1 182 881 - 1 351 857

The work you performed was only on small genome portion (1,3 Mbp). That gives you a flavour of the time cost to do a manual curation on a small genome, and an idea of the amount of work needed to manually curate a big genome (>1 Gbp).

## Check

Before the end of this practical session we will load the reference annotation of drosophila melanogaster allowing you to check your manual annotation. You should just refresh your web page to display this new track.  
Do not be disappointed if your annotation differs a lot from the reference one. Keep in mind that the reference annotation has been curated by experienced experts, and by using more complete evidence.
