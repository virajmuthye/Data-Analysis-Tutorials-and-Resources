# Benchmarking Variant Calls with hap.py

**Author:** Viraj Rajendra Muthye  
**Email:** viraj.muthye@gmail.com

---

## Table of Contents

1. [Introduction](#introduction)
2. [Why Benchmark Variant Calls?](#why-benchmark-variant-calls)
3. [Installation](#installation)
4. [Required Inputs](#required-inputs)
5. [Setting Up the Comparison Engine (vcfeval)](#setting-up-the-comparison-engine-vcfeval)
6. [Running a Basic hap.py Comparison](#running-a-basic-happy-comparison)
7. [Understanding the Output Files](#understanding-the-output-files)
8. [Interpreting Precision, Recall, and F1](#interpreting-precision-recall-and-f1)
9. [Stratifying Results by Region Type](#stratifying-results-by-region-type)
10. [Generating and Plotting ROC Curves](#generating-and-plotting-roc-curves)
11. [Comparing Multiple Callers](#comparing-multiple-callers)
12. [Running hap.py on an HPC Cluster](#running-happy-on-an-hpc-cluster)
13. [Common Pitfalls and Troubleshooting](#common-pitfalls-and-troubleshooting)
14. [Practice Exercise](#practice-exercise)
15. [Resources and References](#resources-and-references)

---

## Introduction

[hap.py](https://github.com/Illumina/hap.py) ("haplotype comparison tool") is the field-standard tool for benchmarking small variant calls (SNPs and indels) from a VCF against a truth set. It was developed at Illumina and is the reference implementation used by the [Global Alliance for Genomics and Health (GA4GH) Benchmarking Team](https://github.com/ga4gh/benchmarking-tools) to standardize how variant calling pipelines are evaluated.

Rather than doing a naive line-by-line VCF diff (which fails badly on indels and multi-allelic sites represented differently by different callers), hap.py normalizes, decomposes, and haplotype-matches variants before comparing them. This makes it possible to fairly compare a query VCF against a truth VCF even when the two represent the same biological variant in different ways.

### What You'll Need

- A Linux/HPC environment (see the [UNIX basics tutorial](UNIX_basic_commands.md) if you're new to the command line)
- `conda`/`mamba` or Docker
- A truth VCF + confident region BED (e.g., from Genome in a Bottle)
- A query VCF (the output of your variant caller)
- The reference genome FASTA the calls were made against

---

## Why Benchmark Variant Calls?

Every step of a variant calling pipeline — aligner, caller, filtering thresholds, ploidy assumptions — affects which variants get called. Benchmarking against a **truth set** (a sample with an independently validated, high-confidence set of variant calls) answers concrete questions:

- Did switching aligners increase indel recall?
- Does a new filtering threshold trade recall for precision?
- How does my pipeline perform in difficult regions (segmental duplications, low-complexity DNA, homopolymers)?
- Is caller A actually better than caller B, or just different?

The [Genome in a Bottle (GIAB) Consortium](https://www.nist.gov/programs-projects/genome-bottle) publishes truth sets for several well-characterized human samples (HG001–HG007) built from multiple sequencing technologies and callers, with confident regions where the truth calls can be trusted. These are the de facto standard truth sets used with hap.py.

---

## Installation

### Option 1: Conda/Mamba (Recommended)

hap.py has an older Python 2 dependency tree, so it's best installed in its own environment:

```bash
mamba create -n happy -c bioconda -c conda-forge hap.py
conda activate happy

# Verify installation
hap.py --version
```

### Option 2: Docker

If your dependencies conflict, Docker sidesteps installation issues entirely:

```bash
docker pull pkrusche/hap.py

docker run -it -v /path/to/data:/data pkrusche/hap.py \
    /opt/hap.py/bin/hap.py --version
```

### Option 3: RTG Tools (for the vcfeval comparison engine)

hap.py's default comparison engine (`xcmp`) works out of the box, but the **`vcfeval` engine (from RTG Tools) is strongly recommended** — it's the GA4GH-endorsed comparison method and handles complex variant representations more robustly.

```bash
mamba install -c bioconda rtg-tools

# Verify
rtg version
```

---

## Required Inputs

| Input | Description | Example Source |
|---|---|---|
| Truth VCF | High-confidence variant calls for the sample | GIAB FTP (e.g., `HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz`) |
| Confident regions BED | Regions where the truth calls are considered reliable | GIAB FTP (e.g., `HG002_GRCh38_1_22_v4.2.1_benchmark_noinbed.bed`) |
| Query VCF | Output of your variant calling pipeline, on the **same sample and reference build** | Your pipeline |
| Reference FASTA | The exact reference genome both VCFs are called against, indexed | e.g., GRCh38 with `.fai` |

> **Critical:** the truth VCF, query VCF, and reference FASTA must all use the **same genome build** (GRCh37 vs. GRCh38) and the **same chromosome naming convention** (`chr1` vs. `1`). Mismatches here are the single most common source of confusing hap.py results (see [Troubleshooting](#common-pitfalls-and-troubleshooting)).

### Downloading a GIAB Truth Set (HG002, GRCh38)

```bash
mkdir -p truth && cd truth

wget https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG002_NA24385_son/NISTv4.2.1/GRCh38/HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz
wget https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG002_NA24385_son/NISTv4.2.1/GRCh38/HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz.tbi
wget https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG002_NA24385_son/NISTv4.2.1/GRCh38/HG002_GRCh38_1_22_v4.2.1_benchmark_noinbed.bed

cd ..
```

### Preparing the Reference

```bash
samtools faidx GRCh38.fasta
```

---

## Setting Up the Comparison Engine (vcfeval)

`vcfeval` requires the reference genome in RTG's **SDF** (Sequence Data File) format. Build this once per reference and reuse it for every comparison:

```bash
rtg format -o GRCh38.sdf GRCh38.fasta
```

This creates a `GRCh38.sdf/` directory. Keep it around — you'll pass its path to every hap.py run.

---

## Running a Basic hap.py Comparison

```bash
hap.py \
    truth/HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz \
    my_pipeline/HG002.query.vcf.gz \
    -f truth/HG002_GRCh38_1_22_v4.2.1_benchmark_noinbed.bed \
    -r GRCh38.fasta \
    -o results/HG002_benchmark \
    --engine=vcfeval \
    --engine-vcfeval-template GRCh38.sdf \
    --threads 8
```

**Argument breakdown:**

| Flag | Meaning |
|---|---|
| (positional 1) | Truth VCF |
| (positional 2) | Query VCF (your caller's output) |
| `-f` | Confident regions BED — restricts truth evaluation to reliable regions |
| `-r` | Reference FASTA |
| `-o` | Output prefix (hap.py writes several files with this prefix) |
| `--engine=vcfeval` | Use RTG's vcfeval for variant matching (recommended over default `xcmp`) |
| `--engine-vcfeval-template` | Path to the SDF built from the reference |
| `--threads` | Parallelize where possible |

Other commonly used flags:

```bash
--pass-only          # Only evaluate variants with FILTER=PASS in the query VCF
--roc QUAL           # Generate ROC/precision-recall data using the QUAL field
--target-regions BED  # Restrict comparison to a target region (e.g., exome capture kit)
--stratification FILE # Break down results by region type (see below)
```

---

## Understanding the Output Files

A run with `-o results/HG002_benchmark` produces:

| File | Contents |
|---|---|
| `HG002_benchmark.summary.csv` | Top-line precision/recall/F1 by variant type (SNP/INDEL) and filter (ALL/PASS) — **start here** |
| `HG002_benchmark.extended.csv` | Detailed breakdown by subtype (e.g., transitions vs. transversions, indel length, genotype match) |
| `HG002_benchmark.vcf.gz` | Merged VCF annotated with TP/FP/FN classification per variant — useful for manually inspecting specific errors |
| `HG002_benchmark.runinfo.json` | Run metadata: hap.py version, command line, timing |
| `HG002_benchmark.roc.all.csv.gz` / `.roc.Locations.*.csv.gz` | ROC data if `--roc` was used |

### Reading `summary.csv`

```
Type,Filter,TRUTH.TOTAL,TRUTH.TP,TRUTH.FN,QUERY.TOTAL,QUERY.FP,QUERY.UNK,METRIC.Recall,METRIC.Precision,METRIC.F1_Score
INDEL,ALL,504501,502691,1810,...
INDEL,PASS,504501,498104,6397,...
SNP,ALL,3327496,3320456,7040,...
SNP,PASS,3327496,3315928,11568,...
```

- **Type**: `SNP` or `INDEL`
- **Filter**: `ALL` (every variant) or `PASS` (only variants passing your caller's filters — this is usually the number you actually care about)
- **TRUTH.TP / TRUTH.FN**: true positives and false negatives, counted against the truth set
- **QUERY.FP**: false positives — variants your caller called that aren't in the truth set (within confident regions)
- **METRIC.Recall / METRIC.Precision / METRIC.F1_Score**: the headline benchmarking metrics

---

## Interpreting Precision, Recall, and F1

$$
\text{Recall} = \frac{TP}{TP + FN} \qquad
\text{Precision} = \frac{TP}{TP + FP} \qquad
F_1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}
$$

- **Low recall, high precision**: your caller is conservative — it's missing real variants but rarely calling wrong ones. Common with aggressive filtering.
- **High recall, low precision**: your caller is calling too much — it finds most true variants but also many false ones. Common with permissive filtering or noisy data.
- **SNP vs. INDEL**: indel calling is intrinsically harder than SNP calling (alignment ambiguity around insertions/deletions), so expect lower indel recall/precision than SNP metrics even from a good pipeline.
- Always compare `PASS` filter rows across pipelines/settings, since `ALL` includes variants you'd never report anyway.

---

## Stratifying Results by Region Type

Aggregate genome-wide metrics can hide the fact that a caller performs badly in specific region types (segmental duplications, low-complexity/homopolymer regions, etc.). GA4GH publishes standardized **stratification BED files** for this purpose.

```bash
# Download GA4GH stratification regions (GRCh38 example)
wget -r -np -nH --cut-dirs=7 \
    ftp://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/genome-stratifications/v3.1/GRCh38/

# stratification.tsv: two columns — <label>\t<path to BED>
```

Run hap.py with the stratification file:

```bash
hap.py \
    truth/HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz \
    my_pipeline/HG002.query.vcf.gz \
    -f truth/HG002_GRCh38_1_22_v4.2.1_benchmark_noinbed.bed \
    -r GRCh38.fasta \
    -o results/HG002_stratified \
    --engine=vcfeval \
    --engine-vcfeval-template GRCh38.sdf \
    --stratification stratification.tsv \
    --threads 8
```

The `extended.csv` output will now include per-stratum rows (e.g., `Subset=alllowmapqualitiy`, `Subset=segdup`), letting you see exactly where your pipeline underperforms.

---

## Generating and Plotting ROC Curves

Adding `--roc QUAL` sweeps the query's QUAL threshold and reports precision/recall at each cutoff — useful for choosing a filtering threshold:

```bash
hap.py \
    truth/HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz \
    my_pipeline/HG002.query.vcf.gz \
    -f truth/HG002_GRCh38_1_22_v4.2.1_benchmark_noinbed.bed \
    -r GRCh38.fasta \
    -o results/HG002_roc \
    --engine=vcfeval \
    --engine-vcfeval-template GRCh38.sdf \
    --roc QUAL \
    --threads 8

# Decompress the ROC table
zcat results/HG002_roc.roc.all.csv.gz > roc_data.csv
```

Plot it with the [ggplot2 tutorial](ggplot2_tutorial.md) techniques covered elsewhere in this repo:

```r
library(ggplot2)
library(dplyr)
library(readr)

roc_data <- read_csv("roc_data.csv") %>%
  filter(Type == "SNP", Filter == "PASS")

ggplot(roc_data, aes(x = METRIC.Recall, y = METRIC.Precision)) +
  geom_line(color = "#2E86AB", linewidth = 1.2) +
  geom_point(size = 1, alpha = 0.5) +
  labs(title = "SNP Precision-Recall Curve",
       subtitle = "HG002 benchmark, QUAL threshold sweep",
       x = "Recall", y = "Precision") +
  theme_minimal()
```

---

## Comparing Multiple Callers

To compare callers side by side, run hap.py once per caller against the same truth set, then combine the `summary.csv` files:

```bash
for caller in deepvariant gatk_haplotypecaller freebayes; do
    hap.py \
        truth/HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz \
        calls/${caller}.HG002.vcf.gz \
        -f truth/HG002_GRCh38_1_22_v4.2.1_benchmark_noinbed.bed \
        -r GRCh38.fasta \
        -o results/${caller} \
        --engine=vcfeval \
        --engine-vcfeval-template GRCh38.sdf \
        --threads 8
done

# Combine summaries, tagging each with its caller name
for caller in deepvariant gatk_haplotypecaller freebayes; do
    awk -v c="$caller" 'NR==1 && FNR==1 {print "Caller," $0} FNR>1 {print c "," $0}' \
        results/${caller}.summary.csv
done > combined_summary.csv
```

This `combined_summary.csv` can then be loaded into R/ggplot2 for a grouped bar chart comparing F1 scores across callers.

---

## Running hap.py on an HPC Cluster

Benchmark runs on whole-genome VCFs can take significant time and memory. See the [UNIX basics tutorial](UNIX_basic_commands.md#working-with-slurm-job-scheduler) for a general SLURM primer. A typical job script:

```bash
#!/bin/bash
#SBATCH --job-name=happy_benchmark
#SBATCH --output=happy_%j.out
#SBATCH --error=happy_%j.err
#SBATCH --time=04:00:00
#SBATCH --ntasks-per-node=8
#SBATCH --mem=32GB

module load anaconda3
source activate happy

hap.py \
    truth/HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz \
    my_pipeline/HG002.query.vcf.gz \
    -f truth/HG002_GRCh38_1_22_v4.2.1_benchmark_noinbed.bed \
    -r GRCh38.fasta \
    -o results/HG002_benchmark \
    --engine=vcfeval \
    --engine-vcfeval-template GRCh38.sdf \
    --threads 8

echo "hap.py finished at $(date)"
```

For interactively monitoring a long benchmark run without staying tied to a single SSH connection, use [tmux](tmux_tutorial.md) — start the run in a named session, detach, and reattach later to check progress.

---

## Common Pitfalls and Troubleshooting

### Chromosome naming mismatch (`chr1` vs `1`)

hap.py will run without error but report suspiciously low recall/precision if the truth VCF, query VCF, and reference use inconsistent contig naming.

```bash
# Check naming convention in each file
zcat truth.vcf.gz | grep -v "^#" | cut -f1 | sort -u | head
zcat query.vcf.gz | grep -v "^#" | cut -f1 | sort -u | head
grep ">" reference.fasta | head
```

Fix with `bcftools annotate --rename-chrs` if needed.

### Genome build mismatch (GRCh37 vs. GRCh38)

Calling against GRCh38 and benchmarking against a GRCh37 truth set will silently produce garbage results — coordinates don't correspond. Always confirm the truth set build matches your pipeline's reference.

### Unsorted or unindexed VCFs

```bash
bcftools sort input.vcf.gz -Oz -o sorted.vcf.gz
tabix -p vcf sorted.vcf.gz
```

### Missing reference `.fai` index

```bash
samtools faidx reference.fasta
```

### Very low recall despite a "good" caller

Check whether you're comparing `PASS`-filtered variants (`Filter=PASS` row) rather than `ALL` — and check the confident regions BED is for the correct truth-set version. GIAB periodically revises confident regions; an outdated BED can be badly out of sync with the truth VCF.

### Memory errors on whole-genome runs

Restrict to specific chromosomes for testing with `-l chr20` before committing to a full genome-wide run.

---

## Practice Exercise

```bash
# 1. Download GIAB HG002 chr20-only truth set and confident regions (smaller, faster to test with)
# 2. Build the RTG SDF for your reference
rtg format -o GRCh38.sdf GRCh38.fasta

# 3. Run hap.py restricted to chr20
hap.py \
    truth/HG002_GRCh38_1_22_v4.2.1_benchmark.vcf.gz \
    my_pipeline/HG002.query.vcf.gz \
    -f truth/HG002_GRCh38_1_22_v4.2.1_benchmark_noinbed.bed \
    -r GRCh38.fasta \
    -o practice/HG002_chr20 \
    -l chr20 \
    --engine=vcfeval \
    --engine-vcfeval-template GRCh38.sdf

# 4. Open practice/HG002_chr20.summary.csv and answer:
#    - What is the SNP recall and precision at PASS?
#    - What is the INDEL recall and precision at PASS?
#    - Which variant type performs worse, and why might that be?
```

---

## Resources and References

### Official Documentation
- [hap.py GitHub Repository](https://github.com/Illumina/hap.py)
- [GA4GH Benchmarking Team](https://github.com/ga4gh/benchmarking-tools)
- [RTG Tools Documentation](https://github.com/RealTimeGenomics/rtg-tools)

### Truth Sets and Stratifications
- [Genome in a Bottle (GIAB) FTP](https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/)
- [GIAB GitHub](https://github.com/genome-in-a-bottle)
- [GA4GH Genome Stratifications](https://github.com/genome-in-a-bottle/genome-stratifications)

### Papers
- Krusche, P. et al. "Best practices for benchmarking germline small-variant calls in human genomes." *Nature Biotechnology* (2019).
- Zook, J.M. et al. "Extensive sequencing of seven human genomes to characterize benchmark reference materials." *Scientific Data* (2016).

### Related Tutorials in This Repository
- [UNIX Basics](UNIX_basic_commands.md) — command line and SLURM fundamentals
- [tmux Tutorial](tmux_tutorial.md) — keeping long benchmark runs alive
- [ggplot2 Tutorial](ggplot2_tutorial.md) — plotting ROC/precision-recall curves

---

For questions, suggestions, or contributions, please contact:
**Viraj Rajendra Muthye** - viraj.muthye@gmail.com

---

*Happy benchmarking!*
