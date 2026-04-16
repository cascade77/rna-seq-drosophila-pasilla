# RNA-seq Analysis, Drosophila melanogaster

Reference-based RNA-seq analysis of *Drosophila melanogaster* to identify differentially expressed genes following *pasilla* gene knockdown, following the GTN tutorial on Galaxy.

## Overview

This repository documents a complete reference-based RNA-seq pipeline run on *Drosophila melanogaster*, carried out as part of the Galaxy Training Network tutorial. The goal was to identify genes that are differentially expressed between *pasilla*-depleted samples and untreated controls, starting from raw paired-end and single-end RNA-seq reads all the way through to a final list of significant DE genes and a volcano plot.

The *pasilla* gene encodes a RNA-binding protein involved in the regulation of splicing. The dataset used here comes from the original Brooks et al. (2011) study, which was one of the first papers to apply RNA-seq to characterise splicing regulation in *Drosophila*.

All steps were run on [usegalaxy.eu](https://usegalaxy.eu) .
## Repository Structure


```
.
├── images/                          # all result figures exported from Galaxy
│   ├── pca_plot.jpg                 # PCA of samples by treatment group
│   ├── sample_distance_heatmap.jpg  # hierarchical clustering of sample-to-sample distances
│   ├── dispersion_estimates.jpg     # gene-wise dispersion estimates from DESeq2
│   ├── pvalue_histogram.jpg         # histogram of raw p-values across all genes
│   ├── ma_plot.jpg                  # MA plot: log fold change vs mean expression
│   └── volcano_plot.jpg             # volcano plot: fold change vs significance
├── results/
│   ├── qc/                          # MultiQC HTML reports for each pipeline stage
│   │   ├── multiqc_falco.html
│   │   ├── multiqc_cutadapt.html
│   │   ├── multiqc_star.html
│   │   └── multiqc_featurecounts.html
│   ├── deseq2/                      # all DESeq2 output tables
│   │   ├── deseq2_results.tabular        # full results: all genes with LFC, p-value, padj
│   │   ├── deseq2_plots.pdf              # original DESeq2 diagnostic plots PDF
│   │   ├── normalized_counts.tabular     # DESeq2 size-factor normalised count matrix
│   │   ├── deseq2_annotated.tabular      # results table with gene names added
│   │   ├── deseq2_significant.tabular    # filtered to padj < 0.05 only
│   │   └── deseq2_significant_cut.tabular# significant genes, selected columns only
│   └── figures/                     # exported figure files
│       └── volcano_plot.pdf              # original volcano plot PDF from Galaxy
├── methods.md                       # step-by-step methods with tool versions and parameters
└── README.md

```

## Pipeline

```
Raw reads (paired-end + single-end FASTQ)
|
Falco — fast quality assessment on raw reads
|
MultiQC — aggregates Falco reports across all samples
|
Cutadapt — trims adapter sequences and low-quality bases
|
MultiQC — aggregates Cutadapt trimming reports
|
RNA STAR — splicing-aware alignment to the Drosophila
reference genome (dm6); produces BAM files
|
MultiQC — aggregates STAR alignment logs
|
featureCounts — counts reads mapping to each gene
using the dm6 GTF annotation
|
MultiQC — aggregates featureCounts summary stats
|
DESeq2 — normalises counts and identifies differentially
expressed genes between pasilla-depleted
and control conditions
|
Annotate DESeq2 output — adds gene names and descriptions
from a reference annotation table
|
Filter (padj < 0.05) — retains only statistically
significant DE genes
|
Volcano Plot — visualises log2 fold change vs
-log10(adjusted p-value) across all genes

```

## Species

| | |
|---|---|
| Species | *Drosophila melanogaster* |
| Genome build | dm6 |
| Condition tested | *pasilla* knockdown vs untreated control |
| Library types | Paired-end + single-end Illumina RNA-seq |

## Data

All input data was obtained from Zenodo as part of the GTN tutorial dataset.

| Dataset | Description |
|---|---|
| 3 treated samples | *pasilla*-depleted RNA-seq reads |
| 4 untreated samples | control RNA-seq reads |

Source: [doi.org/10.5281/zenodo.1185122](https://doi.org/10.5281/zenodo.1185122)

## Tools

| Tool | Version | Purpose |
|---|---|---|
| Falco | 1.2.1+galaxy1 | Raw read QC |
| Cutadapt | 4.9+galaxy1 | Adapter trimming |
| RNA STAR | 2.7.11a+galaxy0 | Spliced read alignment |
| featureCounts | 2.0.6+galaxy0 | Read quantification |
| MultiQC | 1.25.1+galaxy3 | QC report aggregation |
| DESeq2 | 2.11.40.8+galaxy0 | Differential expression |
| Volcano Plot | 0.0.5 | Visualisation |

## Results

| Metric | Value |
|---|---|
| Samples | 7 (3 treated, 4 untreated) |
| DE genes (padj < 0.05) | see `deseq2_significant.tabular` |
| Full results | see `deseq2_results.tabular` |
| Volcano plot | see `results/figures/volcano_plot.pdf` |


<table>
  <tr>
    <td align="center"><img src="images/pca_plot.jpg"/><br/>PCA</td>
    <td align="center"><img src="images/sample_distance_heatmap.jpg"/><br/>Sample Distance Heatmap</td>
  </tr>
  <tr>
    <td align="center"><img src="images/dispersion_estimates.jpg"/><br/>Dispersion Estimates</td>
    <td align="center"><img src="images/pvalue_histogram.jpg"/><br/>p-value Histogram</td>
  </tr>
  <tr>
    <td align="center"><img src="images/ma_plot.jpg"/><br/>MA Plot</td>
    <td align="center"><img src="images/volcano_plot.jpg"/><br/>Volcano Plot</td>
  </tr>
</table>


## Notes
- Tool versions shown are approximate; confirm exact versions from your Galaxy history if needed

## Platform

All steps were run on Galaxy ([usegalaxy.eu](https://usegalaxy.eu)).

## Citations

- Brooks et al. (2011) Conservation of an RNA regulatory map between *Drosophila* and mammals. *Genome Research*. https://doi.org/10.1101/gr.108662.110
- Dobin et al. (2013) STAR: ultrafast universal RNA-seq aligner. *Bioinformatics*. https://doi.org/10.1093/bioinformatics/bts635
- Liao et al. (2014) featureCounts: an efficient general purpose program for assigning sequence reads to genomic features. *Bioinformatics*. https://doi.org/10.1093/bioinformatics/btt656
- Love et al. (2014) Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2. *Genome Biology*. https://doi.org/10.1186/s13059-014-0550-8
- Martin (2011) Cutadapt removes adapter sequences from high-throughput sequencing reads. *EMBnet.journal*. https://doi.org/10.14806/ej.17.1.200

## Reference

GTN Tutorial: [Reference-based RNA-Seq data analysis](https://training.galaxyproject.org/training-material/topics/transcriptomics/tutorials/ref-based/tutorial.html)
