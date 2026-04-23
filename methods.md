# Methods

All analysis was performed on [usegalaxy.eu](https://usegalaxy.eu) following the Galaxy Training Network reference-based RNA-seq tutorial. Input data consisted of 7 RNA-seq samples from the Brooks et al. (2011) *pasilla* knockdown study in *Drosophila melanogaster* (3 *pasilla*-depleted, 4 untreated controls), obtained from Zenodo (doi.org/10.5281/zenodo.6457007). The 7 samples include both paired-end and single-end libraries, reflecting the original study design. All raw data is publicly available from NCBI GEO under accession GSE18508.

## 1. Quality Control

Raw reads were assessed using **Falco** (v1.2.4+galaxy0), a fast reimplementation of FastQC optimised for large collections. Because the input data was organised as a list of paired collections, the collection was first flattened into a simple list using the Flatten Collection tool before running Falco, as MultiQC does not support list-of-pairs collections directly. Falco was run on all 4 FASTQ files (2 paired-end samples, 2 reads each) and produced per-sample reports covering per-base sequence quality scores, per-sequence quality score distributions, per-base GC content, sequence duplication levels, and adapter contamination.

Reports from all samples were aggregated into a single interactive HTML using **MultiQC** (v1.27+galaxy4) with the FastQC module selected (Falco output is fully compatible with the FastQC parser). Review of the MultiQC report showed that read quality was generally good across all samples, with a slight quality drop at the 3' end of reads, most pronounced in the reverse reads of the treated paired-end sample. No severe adapter contamination was observed, but quality trimming was applied regardless as standard practice.

## 2. Adapter Trimming

Reads were trimmed using **Cutadapt** (v5.2+galaxy0) to remove adapter sequences and low-quality bases from the 3' end of reads. Paired-end samples were processed as paired collections to ensure read pairs were trimmed together, preserving pairing integrity. Reads falling below a minimum length after trimming were discarded. The following parameters were applied:

| Parameter | Value |
|---|---|
| Input mode | Paired-end collection |
| Quality cutoff (R1 and R2) | 20 |
| Minimum read length | 20 bp |
| Adapter detection | Automatic |

A second MultiQC (v1.27+galaxy4) aggregation was run on the Cutadapt reports to summarise trimming efficiency across samples, including the number of read pairs removed due to both reads falling below the length threshold, and the proportion of bases trimmed per sample.

## 3. Alignment

Trimmed reads were aligned to the *Drosophila melanogaster* reference genome (dm6, BDGP Release 6) using **RNA STAR** (v2.7.11b+galaxy0), a two-pass splice-aware aligner. Splice-aware alignment is essential for RNA-seq data from eukaryotic organisms because a substantial proportion of reads span exon-exon junctions and would fail to align with a standard short-read aligner. STAR handles this by performing a seed search phase followed by a stitching phase to reconstruct read-level alignments across splice junctions.

The Ensembl BDGP6.32.109 gene annotation file (UCSC-adapted GTF) was provided to STAR to guide splice junction detection. The junction overhang parameter was set to 36 (read length minus 1). STAR was also configured to output per-gene read counts (GeneCounts mode), though featureCounts was used for the final quantification step. The following key parameters were used:

| Parameter | Value |
|---|---|
| Input mode | Paired-end collection |
| Reference genome | dm6 Full (built-in Galaxy index) |
| Annotation GTF | Drosophila_melanogaster.BDGP6.32.109_UCSC.gtf.gz |
| Genomic sequence overhang | 36 |
| Per gene output | GeneCounts |
| Coverage output | Yes, bedgraph format |

STAR alignment logs were aggregated with a third MultiQC (v1.27+galaxy4) run. Alignment rates of >79% uniquely mapped reads were observed across all samples, which is acceptable for this dataset. Rates below 70% would warrant investigation for contamination or annotation mismatch.

## 4. Read Quantification

Aligned reads were assigned to genomic features using **featureCounts** (v2.1.1+galaxy0), part of the Subread package. featureCounts counts the number of reads (or read pairs for paired-end data) that overlap each annotated gene in the dm6 GTF. The tool was run in its default union overlap mode, which assigns a read to a gene if it overlaps any part of that gene's exons and does not overlap any other gene. Reads mapping to multiple genes (ambiguous) or to no annotated feature are reported separately in the summary.

featureCounts produces two outputs used in downstream steps: a count matrix (one row per gene, one column per sample) and a feature lengths file (the total exonic length of each gene), which is needed for length-bias correction in GO enrichment analysis. A fourth MultiQC (v1.27+galaxy4) aggregation was run on the featureCounts summary files to inspect the proportion of assigned reads across all samples.

## 5. Differential Expression Analysis

Differential expression was analysed using **DESeq2** (v2.11.49.8+galaxy2). DESeq2 models RNA-seq count data using a negative binomial distribution and applies a median-of-ratios normalisation to account for differences in sequencing depth and library composition between samples. Gene length normalisation is not required in a between-sample comparison for the same gene.

Because the dataset contains both paired-end and single-end libraries, a two-factor design was used with treatment condition as the primary factor of interest and sequencing type as a covariate. This allows DESeq2 to estimate the effect of *pasilla* depletion while controlling for systematic differences between library types, producing more accurate fold change estimates.

| Factor | Levels |
|---|---|
| Treatment (primary) | treated (*pasilla* depleted), untreated |
| Sequencing type (covariate) | paired-end, single-end |

DESeq2 outputs used in this analysis:

| Output | Description |
|---|---|
| Results table | All genes with log2 fold change, p-value, Benjamini-Hochberg adjusted p-value |
| Normalised counts | Size-factor normalised count matrix for all 7 samples |
| Diagnostic plots PDF | MA plot, PCA, sample-to-sample distance heatmap, dispersion estimates, p-value histogram |

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
</table>

![MA Plot](images/ma_plot.jpg) 


The Wald test was used to test for differential expression between treated and untreated conditions. P-values were corrected for multiple testing using the Benjamini-Hochberg procedure. Genes with a missing adjusted p-value (due to low counts or extreme outliers) were excluded from downstream filtering.

## 6. Annotation

The DESeq2 results table was annotated using the **Annotate DESeq2/DEXSeq output tables** tool (v1.1.0+galaxy1), which joins the results table with a *Drosophila melanogaster* reference annotation table to append gene names and functional descriptions alongside the statistical results. This makes the output table interpretable in biological terms rather than Ensembl gene IDs alone.

## 7. Filtering

The annotated results table was filtered to retain only genes with an adjusted p-value (padj) below 0.05, using the Filter tool with the expression `c7 < 0.05` on the padj column. This threshold corresponds to a 5% false discovery rate under the Benjamini-Hochberg correction. The filtered output represents the set of statistically significant differentially expressed genes, which is the primary result of the analysis.

A second Cut step was applied to extract the gene ID, adjusted p-value, and log2 fold change columns from the filtered table into a clean two-column format for downstream use.

## 8. Visualisation

A volcano plot was generated using the **Volcano Plot** tool to visualise the full distribution of DE results. The volcano plot displays log2 fold change on the x-axis and negative log10 of the adjusted p-value on the y-axis for all tested genes, allowing the significant and biologically meaningful DE genes to be identified at a glance. Genes in the upper-left and upper-right quadrants represent strongly and significantly down-regulated and up-regulated genes respectively.

![Volcano Plot](images/volcano_plot.jpg) 

## 9. GO Enrichment Analysis

Gene Ontology enrichment analysis using **goseq** was planned as the final step to identify biological processes, molecular functions, and cellular components enriched among the differentially expressed genes. goseq accounts for length bias in RNA-seq data, which arises because longer genes are more likely to be detected as differentially expressed simply due to having more reads, not because of genuine biological signal.

The goseq input requires a two-column file containing all tested gene IDs and a boolean column (True/False) indicating whether each gene is differentially expressed. This boolean column was to be generated using the **Compute an expression on every row** tool with the expression `bool(float(c7)<0.05)` applied to the DESeq2 results table.
