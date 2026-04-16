# Methods

All analysis was performed on [usegalaxy.eu](https://usegalaxy.eu) following the Galaxy Training Network reference-based RNA-seq tutorial. Input data consisted of 7 RNA-seq samples from the Brooks et al. (2011) *pasilla* knockdown study in *Drosophila melanogaster* (3 *pasilla*-depleted, 4 untreated controls), obtained from Zenodo.

## Quality Control

Raw reads were assessed with **Falco** (v1.2.1+galaxy1). Reports from all samples were aggregated using **MultiQC** (v1.25.1+galaxy3) to get an overview of per-base quality scores, adapter content, and duplication levels across the dataset.

## Adapter Trimming

Reads were trimmed using **Cutadapt** (v4.9+galaxy1) to remove adapter sequences and low-quality bases from the 3' ends. Trimming quality was reviewed through a second MultiQC aggregation of Cutadapt reports.

## Alignment

Trimmed reads were aligned to the *Drosophila melanogaster* reference genome (dm6) using **RNA STAR** (v2.7.11a+galaxy0), a splicing-aware aligner. STAR alignment logs were aggregated with MultiQC to assess mapping rates across all samples.

## Read Quantification

Aligned reads were assigned to genomic features using **featureCounts** (v2.0.6+galaxy0) with the dm6 GTF annotation. featureCounts summary statistics were reviewed through a final MultiQC report.

## Differential Expression Analysis

Differential expression was analysed using **DESeq2** (v2.11.40.8+galaxy0), comparing *pasilla*-depleted samples against untreated controls. DESeq2 outputs include a full results table with log2 fold changes, p-values, and adjusted p-values (Benjamini-Hochberg), normalised count matrices, and diagnostic plots (MA plot, PCA, sample distance heatmap).

The DESeq2 results table was annotated with gene names and descriptions using a *Drosophila* reference annotation. Genes with adjusted p-value < 0.05 were retained as statistically significant differentially expressed genes.

## Visualisation

A volcano plot was generated to visualise the relationship between log2 fold change and statistical significance (-log10 adjusted p-value) across all tested genes.

## GO Enrichment

GO enrichment analysis using **goseq** was attempted but could not be completed due to a tool-level error on usegalaxy.eu in the Compute step required to prepare the input.
