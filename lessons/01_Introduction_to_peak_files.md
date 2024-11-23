---
title: "Intro to dataset and peak call file formats"
author: "Heather Wick, Upendra Bhattarai, Meeta Mistry, Will Gammerdinger"
date: "Aug 13th, 2024"
---

Contributors: Heather Wick, Upendra Bhattarai, Meeta Mistry, Will Gammerdinger

Approximate time: 40 minutes

## Learning Objectives

In this lesson, we will:

* Explain the dataset and the biological context
* Define peaks as genomic coordinate data
* Describe file formats for peak data

## Introduction to the dataset

For this workshop we will be working with ChIP-seq data from a publication in Neuron by *Baizabal et al., 2018* [1](https://pmc.ncbi.nlm.nih.gov/articles/PMC6667181/).

> Please note that even though we are utilizing a ChIP-seq dataset for histone mark in this workshop, some of these steps in the workflow can very similarly be used for other peak data such as ATAC-seq or CUT&RUN.

*Baizabal et al.,* sought to understand how chromatin-modifying enzymes function in neural stem cells to establish the epigenetic landscape that determines cell type and stage-specific gene expression. Chromatin-modifying enzymes are transcriptional regulators that control gene expression through covalent modification of DNA or histones. In this workshop, we will be evaluating ***H3K27Ac*** (Histone 3 Lysine 27 Acetylation) modifications, which are post-translational modifications associated with increased transcriptional activation.

<p align="center">
<img src="../img/05Epigenetics06-edit.png" width="400">
</p>

_Image adapted from: [American Society of Hematology](https://www.hematology.org/research/ash-agenda-for-hematology-research/epigenetic-mechanisms)_

### PRDM16

The transcriptional regulator **PRDM16 is a chromatin-modifying enzyme** that belongs to the larger PRDM (Positive Regulatory Domain) protein family, that is structurally defined by the **presence of a conserved N-terminal histone methyltransferase PR domain** ([Hohenauer and Moore, 2012](https://journals.biologists.com/dev/article/139/13/2267/45169/The-Prdm-family-expanding-roles-in-stem-cells-and)). 

* PRDM16 has been shown to function *in vitro* as a histone 3 lysine 9 (H3K9) and histone 3 lysine 4 (H3K4) mono-methyltransferase ([Pinheiro et al., 2012](https://www.sciencedirect.com/science/article/pii/S0092867412009385), [Zhou et al., 2016](https://www.sciencedirect.com/science/article/pii/S109727651600188X)). 
* PRDM16 also regulates gene expression by forming complexes with transcriptional co-factors and other histone-modifying proteins ([Chi and Cohen, 2016](https://www.sciencedirect.com/science/article/pii/S104327601500226X?casa_token=VOBAb4QhyXgAAAAA:c69XzQwZ86M4BcPt02cNKjn163X5pBZMTQHJX4D2HdMvgO3hrQE7N6L0YmFSWwucs2GhXPhBtw)). 
* PRDM16 was previously shown to control embryonic and post-natal neural stem cell maintenance and differentiation in the brain ([Chuikov et al., 2010](https://www.nature.com/articles/ncb2101), [Inoue et al., 2017](https://journals.biologists.com/dev/article/144/3/385/48274/Prdm16-is-crucial-for-progression-of-the), [Shimada et al., 2017](http://genesdev.cshlp.org/content/31/11/1134.short)). 


**How PRDM16 functions to regulate transcriptional programs in the developing cerebral cortex remains largely unknown.**

In this paper, the authors use various techniques to identify and validate the targets and activities of PRDM16, including ChIP-seq, bulk RNA-seq, FACS, in-situ hybridization and immunofluorescent microscopy on brain samples from embryonic mice and a generation of PRDM16 conditional knockout mice. 

<p align="center">
<img src="../img/graphical_abstract.png" width="500">
</p>


From the RNA-seq data, they found that the absence of PRDM16 in cortical neurons resulted in the misregulation of over a thousand genes during neurogenesis. To identify the subset of genes that are transcriptional targets of PRDM16 and to understand how these genes are directly regulated, they performed chromatin immunoprecipitation followed by sequencing (ChIP-seq).


**Hypothesis:** How does the histone methyltransferase PRDM16 work with other chromatin machinery to either silence or activate expression of sets of genes that impact the organization of the cerebral cortex?

### Setting up 

> Prior to the workshop, we asked that you download and uncompress the R Project that we will be using during this workshop. If you haven't had a chance to do this, please right click [this link](https://www.dropbox.com/scl/fi/q65yhy60q41p0zpv7ffat/Peak_analysis.zip?rlkey=mfzg1bnbjkabox1zwxb7gn9od&st=4xxsrmkd&dl=1) and select **"Save Link As..."** in order to download the compressed R Project to your desired location. Next, double-click on the compressed ZIP file in order to uncompress it. 

1) Once you have uncompressed the R Project, you can double-click "Peak_analysis.Rproj" wihtin the "Peak_analysis" directory to open the R Project. It should look like:

<p align="center">
<img src="../img/R_studio.png" width="800">
</p>

2) Open an R script by clicking **"File"**, then **"New File >"** and selecting **"R script"**.
3) Save the new R script, by clicking **"File"**, and selecting **"Save As..."**
4) Name the file **"peak_analysis.R"** and click **"Save"**

Now your R Studio should look like:

<p align="center">
<img src="../img/R_studio_with_script.png" width="800">
</p>

### Data Management Best Practices

When working with next-generation sequencing data, it can be very enticing to race to investigating your data soon after receiving it. However, it is considered best practice to take a few minutes to set-up an environment that will help keep you and your data organized *before* you start your analysis. The downloaded project already has this done, but we will discuss what we did to help organize ourselves in this section. 

Importantly, data management practices should be applied *throughout* your analyses. Some other advantages to a commiting to strong data management practices are:

- Helping yourself and others if the data needs to be re-analyzed because it is well-organized
- Dessiminating data to collaborators and other researchers is easier
- Getting credit for the data you generated

You can visit the [Harvard's Biomedical Data Management Website](https://datamanagement.hms.harvard.edu/about/what-research-data-management) for more information on research data management suggestions and policies.

<p align="center">
<img src="../img/RDM_lifecycle.png"  width="800">
</p>

_Image acquired from the [Harvard Biomedical Data Management Website](https://datamanagement.hms.harvard.edu/data-lifecycle)_

#### Project Organization

Before we analyze any data, let's orient ourselves with the project that we are working within first. Within this project we currently have 3 directories:

- **data** - This directory contains data that we have pre-processed
- **meta** - This directory is where we will be placing any metadata and also contains a summary file, *metrics.csv*
- **results** - This directory is where we will be placing our figures 

Directories such as these three are a great starting place for many types of analysis. As your data and analyses become more complex or get re-analyzed in different ways, don't be afraid to add new directories or subdirectories to these directories as needed. For example within the **data** directory, we can see there are subdirectories for each software tool used, rather than lumping all of the data together into a single **data** directory. 

#### Documentation

We will also need to add a header to the top of our R Script. This should let future you, as well as, collaboratos know key pieces of information about the script:

- **What** is the name of the script
- **Who** wrote the script
- **When** the script was written
- **Why** the script was written
- **Usage** information that someone using the script should be aware of

Let's create this and add it to the top of our script:

```
# peak_analysis.R
# Written by the Harvard Chan Bioinformatics Core on November 15th, 2024
# This script was written as a demo for the Peak Analysis Workshop
# In order to use this script, the user will need to have downloaded and uncompress the R project from https://www.dropbox.com/scl/fi/q65yhy60q41p0zpv7ffat/Peak_analysis.zip?rlkey=mfzg1bnbjkabox1zwxb7gn9od&st=4xxsrmkd&dl=1
```

## What is a peak?
A peak represents a region of the genome which was found to be bound to the protein or histone modification of choice. Chromatin Immunoprecipitation followed by sequencing (ChIP-seq) is a central method in epigenomic research which allows us to query peaks. A typical ChIP-seq workflow is outlined in the image below. 

In ChIP-seq experiments, a transcription factor, cofactor, histone modification, or other chromatin protein of interest is enriched by immunoprecipitation from cross-linked cells, along with its associated DNA. The immunoprecipitated DNA fragments are then sequenced, followed by identification of enriched regions of DNA or peaks using peak-calling software, such as MACS2. These peak calls can then be used to make biological inferences by determining the associated genomic features and/or over-represented sequence motifs.

<p align="center">
<img src="../img/chip_expt_workflow.png" width="400">
</p>

_Image source: ["From DNA to a human: What the ENCODE and Roadmap Epigenome Projects can teach us about how we are who we are"](https://portlandpress.com/biochemist/article/37/5/24/773/From-DNA-to-a-human-What-the-ENCODE-and-Roadmap)_

For more detailed information on the process of going from sequenced reads to peaks, please see the [pre-reading lesson](00a_peak_calling_workflow_review.md). In this workshop, we will describe a range of different analyses that can be done using peaks.


## Peak file formats
In this lesson, we will introduce you to several important file formats that you will encounter when working with peak calls, which follow the **BED format** (**B**rowser **E**xtensible **D**ata). We will also describe the contents of the **narrowPeak** files (output from MACS2) and how it relates to BED. 


### BED

The BED file format is tab-delimited (columns separated by tabs) and contains information about the coordinates for particular genome features.

<p align="center">
<img src="../img/bed.png" width="700">
</p>

**The coordinates in BED files are 0-based**. What does this mean? Among standard file formats, genomic coordinates can be represented in two different ways as shown in the image below. 

* **Zero-based** is shown at the top of the image. This is the preferred format for programmers.
* **One-based** is shown at the bottom. This is more intuitive and generally preferred by biologists. 

<p align="center">
<img src="../img/Interbase.png" width="300">
</p>

Given the example above, **what coordinates would you use to define the sequence `ATG`?** 

* If you were using the the 1-based (bottom) method you would indicate **4 to 6**. 
* Using the 0-based method you would define the range as **3 to 6**. 

The benefits to having a **zero-based system** is the **ease of calculating distance or length** of sequences. We can easily determine the length of the `ATG` sequence using the zero-based coordinates by subtracting the start from the end, whereas for one-based coordinates we would need to add one after the subtraction. Therefore, many file formats used in computation, including **the BED file format**, use zero-based coordinates. 

BED files **require at least 3 fields** indicating the **genomic location of the feature**, including the chromosome along with the start and end coordinates. However, there are 9 additional fields that are optional, as shown in the image below.

<p align="center">
<img src="../img/bed_file.png" width="800">
</p>

### narrowPeak

A narrowPeak (.narrowPeak) file is used by the ENCODE project to provide called peaks of signal enrichment based on pooled, normalized (interpreted) data. The narrowPeak file is a BED6+4 format, which means the first 6 columns of a standard BED file  with **4 additional fields**:

<p align="center">
<img src="../img/narrowPeak.png"  width="800">
</p>

Each row in the narrowPeak file represents a called peak. Below is an the example of a narrowPeak file, displaying the coordinate and statistical information for a handful of called peaks.

<p align="center">
<img src="../img/narrow_peak_example.png">
</p>

### broadPeak

A broadPeak (.broadPeak) file is very similar to a narrowPeak file, but it is BED6+3 because it is missing one element that the narrowPeak format has: the final column, which is the point-source, or summit coordinate, for each peak. This is because broad peaks are large regions rather than sharp peaks; they are called in MACS2 by combining adjacent narrow peaks.

### gappedPeak

A gappedPeak (.gappedPeak) file is a BED12+3 file which contains broad peaks as well as narrow peaks within the broad peaks. You can learn more about this format (and the format of the other files) on the [Encode website](https://genome.ucsc.edu/FAQ/FAQformat.html#format14).

## Workflow

For this analysis, we are going to be analyzing the data through a pre-planned workflow. Below we have summarized this workflow with a schematic:

<p align="center">
<img src="../img/Workflow.png">
</p>

Each step of this workflow is summmarized below:

- **Peak Calls** - Peak call data will be the input data for the analysis. More information on how this can be derived from raw sequence reads, can be found [here](https://hbctraining.github.io/Intro-to-ChIPseq-flipped/).
- **Peak QC** - Quality control that can be applied to the analysis:
  - *QC Metrics Assessment* - Evaluating some important QC metrics to ensure we are using high-quality reads, the reads were aligned to the reference well and the peak metrics are within [ENCODE guidelines](https://www.encodeproject.org/chip-seq/histone/).
  - *Replicate Concordance* - Evaluate concordance of peak calls across replicates
  - *Qualitative Assessment* - **MEETA ADD SOMETHING HERE**
- **Peak Annotation** - Identify transcript models near called peaks
- **Differential Enrichment** - Quantify peak calls which show diffential binding across conditions
- **Peak Visualization** - Inspect the peak calls and differentially called peaks within an [IGV browser](https://igv.org/) to ensure that they qualitatively appear accurate.
- **Motif Discovery** - Identify enriched sequence motifs within peaks
- **Functional Analysis** - Investigate whether certain biological pathways are enriched for peaks or differential binding across conditions

***

[Back to the Schedule](../schedule/README.md) 

[Next lesson >>](02a_peak_quality_metrics_assesment.md)

***

*This lesson has been developed by members of the teaching team at the [Harvard Chan Bioinformatics Core (HBC)](http://bioinformatics.sph.harvard.edu/). These are open access materials distributed under the terms of the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), which permits unrestricted use, distribution, and reproduction in any medium, provided the original author and source are credited.*
