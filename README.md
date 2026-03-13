# RasterMan <img src="man/figures/logo.png" align="right" height="120"/>

> High-performance Manhattan plot visualization for large-scale genomic association studies

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Platform](https://img.shields.io/badge/platform-online-brightgreen)](https://iasbreeding.cn/rasterman/)

---

## Overview

RasterMan uses adaptive C++ rasterization to render Manhattan plots of million-scale GWAS / eQTL datasets with **~18× speedup** and **~60% memory reduction** compared to standard `qqman`. Three usage modes are available:

| Mode | When to use |
|------|-------------|
| **Cloud platform** | Quick exploratory plots, no R installation needed |
| **Rscript CLI** | Batch processing, HPC pipelines, automation |
| **R API** | Integration into your own R scripts and workflows |

---

## Installation (local use)

```r
install.packages("RasterMan_1.0.0.tar.gz", repos = NULL, type = "source")
```

**Dependencies** (installed automatically):

```r
install.packages(c("Rcpp", "ggplot2", "data.table", "dplyr",
                   "magrittr", "ggrepel"))
# Recommended for combined plots:
install.packages("patchwork")
```

---

## Mode 1 — Cloud Platform

**URL:** [https://iasbreeding.cn/rasterman/](https://iasbreeding.cn/rasterman/)

No installation required. Upload your summary statistics file and configure the plot in the browser.

### Workflow

```
① Upload file  →  ② Set column names  →  ③ Configure style  →  ④ Download plot
```

### Supported input formats

| Format | Notes |
|--------|-------|
| Plain text (`.txt`, `.tsv`, `.csv`) | Space / tab / comma delimited |
| Gzip compressed (`.gz`) | Recommended for large files |
| Maximum file size | 500 MB |

### Steps

**① Upload file**
Click **Upload** and select your summary statistics file. The platform auto-detects column names; confirm or correct them in the column mapping panel.

**② Column mapping**

| Field | Description | Common values |
|-------|-------------|---------------|
| Chromosome | Integer chromosome identifier | `Chr`, `CHR`, `CHROM` |
| Position | Base-pair position | `Pos`, `BP`, `POS` |
| P-value | Association p-value | `P`, `Pvalue`, `p_wald` |
| Gene *(cis/trans only)* | Gene name or ID | `Gene`, `gene_name` |
| Gene Chr / Start / End *(trans only)* | Gene genomic coordinates | |
| Effect *(trans only, optional)* | Beta / effect size for colour encoding | `beta`, `slope` |

**③ Style options**

| Option | Values |
|--------|--------|
| Plot type | GWAS Manhattan · Cis-eQTL · Trans-eQTL |
| Color scheme | modern · classic · colorblind · pastel · vibrant · gray |
| Significance threshold | Default 5×10⁻⁸ (GWAS) or 1×10⁻⁵ (eQTL) |
| Lead SNP annotation | Enable/disable, window size |
| Gene annotation | Upload BED file |
| Rasterization method | auto · smart · multilevel · adaptive · basic · off |
| Output format | PNG · TIFF · PDF |
| Resolution (DPI) | 150 / 300 / 600 |

**④ Download**
Click **Generate Plot** → preview → **Download**.

---

## Mode 2 — Rscript Command Line

### Setup

```bash
# Get the path to installed CLI scripts
SCRIPTS=$(Rscript -e "cat(system.file('scripts', package='RasterMan'))")

# Optional: copy to your project directory
cp $SCRIPTS/plot_gwas.R        ./
cp $SCRIPTS/plot_cis_eqtl.R   ./
cp $SCRIPTS/plot_trans_eqtl.R ./
```

---

### GWAS Manhattan plot

```bash
Rscript $SCRIPTS/plot_gwas.R <input_file> [options]
Rscript $SCRIPTS/plot_gwas.R --help
```

**Common examples:**

```bash
# GCTA .mlma output
Rscript $SCRIPTS/plot_gwas.R results.mlma \
    --chr-col Chr --bp-col bp --p-col p

# PLINK .assoc — lead SNP annotation + combined QQ plot
Rscript $SCRIPTS/plot_gwas.R plink.assoc \
    --chr-col CHR --bp-col BP --p-col P --snp-col SNP \
    --anno-snp --window-size 500000 \
    --qq-combine --color-scheme colorblind \
    --format png --dpi 300

# BOLT-LMM — gene annotation
Rscript $SCRIPTS/plot_gwas.R bolt.results \
    --chr-col SNP_CHR --bp-col SNP_BP --p-col P_BOLT_LMM \
    --anno-snp --anno-gene --gene-bed hg38_genes.bed \
    --output-prefix bolt_manhattan

# Publication-quality TIFF
Rscript $SCRIPTS/plot_gwas.R results.mlma \
    --chr-col Chr --bp-col bp --p-col p \
    --raster-method smart --format tiff --dpi 600 \
    --width 14 --height 6
```

**All options:**

| Option | Default | Description |
|--------|---------|-------------|
| `--chr-col NAME` | CHR | Chromosome column |
| `--bp-col NAME` | BP | Position column |
| `--p-col NAME` | P | P-value column |
| `--snp-col NAME` | SNP | SNP ID column |
| `--sig-threshold VALUE` | 5e-8 | Genome-wide significance line |
| `--anno-snp` | off | Annotate lead SNPs |
| `--window-size BP` | 500000 | Clumping window size |
| `--anno-threshold VALUE` | = sig-threshold | Annotation p-value threshold |
| `--max-anno N` | 50 | Max SNP labels |
| `--anno-gene` | off | Annotate nearest gene |
| `--gene-bed FILE` | — | Gene BED file (chr, start, end, name) |
| `--gene-distance BP` | 50000 | Max gene–SNP distance |
| `--qq-plot` | off | Save QQ plot separately |
| `--qq-only` | off | QQ plot only |
| `--qq-combine` | off | Manhattan + QQ in one figure |
| `--color-scheme NAME` | modern | Color scheme (see table below) |
| `--color-even HEX` | — | Custom even-chromosome color |
| `--color-odd HEX` | — | Custom odd-chromosome color |
| `--raster-method METHOD` | auto | Rasterization method |
| `--raster-width N` | 2000 | Grid width |
| `--raster-height N` | 1000 | Grid height |
| `--force-raster` | off | Force rasterization |
| `--no-raster` | off | Disable rasterization |
| `--format FORMAT` | png | Output format (png/tiff/pdf) |
| `--dpi N` | 300 | Resolution |
| `--width INCHES` | 12 | Plot width |
| `--height INCHES` | 6 | Plot height |
| `--output-prefix PREFIX` | gwas_plot | Output filename prefix |
| `--no-background` | off | Remove chromosome background bands |

---

### Cis-eQTL Manhattan plot

```bash
Rscript $SCRIPTS/plot_cis_eqtl.R <input_file> [options]
Rscript $SCRIPTS/plot_cis_eqtl.R --help
```

**Common examples:**

```bash
# Basic usage
Rscript $SCRIPTS/plot_cis_eqtl.R cis.txt.gz \
    --chr-col Chr --bp-col Pos --p-col P --gene-col gene_name

# Large dataset (>10M rows) — use multilevel
Rscript $SCRIPTS/plot_cis_eqtl.R big_cis.txt.gz \
    --chr-col Chr --bp-col Pos --p-col P --gene-col gene_name \
    --raster-method multilevel

# Highlight significant cis-eGenes
Rscript $SCRIPTS/plot_cis_eqtl.R cis.txt.gz \
    --chr-col Chr --bp-col Pos --p-col P --gene-col gene_name \
    --anno-gene sig_genes.txt \
    --highlight-color "#E74C3C" --highlight-size 1.5 \
    --color-scheme colorblind \
    --output cis_manhattan.png --dpi 300
```

**All options:**

| Option | Default | Description |
|--------|---------|-------------|
| `--chr-col NAME` | Chr | Chromosome column |
| `--bp-col NAME` | Position | Position column |
| `--p-col NAME` | Pvalue | P-value column |
| `--gene-col NAME` | Gene | Gene name column |
| `--snp-col NAME` | — | SNP ID column (auto chr:pos if absent) |
| `--sig-threshold VALUE` | 1e-5 | Significance threshold |
| `--min-gene-logp VALUE` | 5 | Min −log₁₀(P) for gene labels |
| `--max-labels N` | all | Max gene labels |
| `--anno-gene FILE` | — | cis-eGene list to highlight (one per line) |
| `--highlight-color COLOR` | red | Highlight point color |
| `--highlight-size SIZE` | 1.5 | Highlight point size |
| `--color-scheme NAME` | modern | Color scheme |
| `--color-even HEX` | — | Custom even-chromosome color |
| `--color-odd HEX` | — | Custom odd-chromosome color |
| `--raster-method METHOD` | auto | Rasterization method |
| `--raster-width N` | auto | Grid width |
| `--raster-height N` | auto | Grid height |
| `--force-raster` | off | Force rasterization |
| `--no-raster` | off | Disable rasterization |
| `--output FILE` | cis_eqtl_manhattan.png | Output filename |
| `--width INCHES` | 14 | Plot width |
| `--height INCHES` | 7 | Plot height |
| `--dpi N` | 300 | Resolution |
| `--no-background` | off | Remove chromosome bands |

---

### Trans-eQTL 2D Manhattan plot

```bash
Rscript $SCRIPTS/plot_trans_eqtl.R <input_file> [options]
Rscript $SCRIPTS/plot_trans_eqtl.R --help
```

**Common examples:**

```bash
# Basic usage (color by -log10P)
Rscript $SCRIPTS/plot_trans_eqtl.R trans.txt.gz \
    --chr-col Chr --pos-col Pos --p-col P \
    --gene-chr-col gene_chr \
    --gene-start-col gene_start \
    --gene-end-col gene_end

# Color by effect direction
Rscript $SCRIPTS/plot_trans_eqtl.R trans.txt.gz \
    --chr-col Chr --pos-col Pos --p-col P \
    --gene-chr-col gene_chr --gene-start-col gene_start --gene-end-col gene_end \
    --slope-col beta --color-scheme modern

# Custom gradient colors
Rscript $SCRIPTS/plot_trans_eqtl.R trans.txt.gz \
    --chr-col Chr --pos-col Pos --p-col P \
    --gene-chr-col gene_chr --gene-start-col gene_start --gene-end-col gene_end \
    --slope-col beta \
    --color-low "#0173B2" --color-high "#DE8F05" \
    --output trans_manhattan.png --dpi 300
```

**All options:**

| Option | Default | Description |
|--------|---------|-------------|
| `--chr-col NAME` | Chr | SNP chromosome column |
| `--pos-col NAME` | Pos | SNP position column |
| `--p-col NAME` | Pvalue | P-value column |
| `--gene-chr-col NAME` | Gene_chr | Gene chromosome column |
| `--gene-start-col NAME` | Start | Gene start position column |
| `--gene-end-col NAME` | End | Gene end position column |
| `--snp-col NAME` | — | SNP ID column (auto chr:pos if absent) |
| `--gene-col NAME` | — | Gene name column (optional) |
| `--slope-col NAME` | — | Beta/effect column for gradient coloring |
| `--color-scheme NAME` | modern | Gradient color scheme |
| `--color-low HEX` | — | Custom negative-effect color |
| `--color-high HEX` | — | Custom positive-effect color |
| `--raster-method METHOD` | auto | Rasterization method (basic/off) |
| `--raster-width N` | 2000 | Grid size |
| `--force-raster` | off | Force rasterization |
| `--no-raster` | off | Disable rasterization |
| `--output FILE` | trans_eqtl_manhattan.png | Output filename |
| `--width INCHES` | 10 | Plot width |
| `--height INCHES` | 10 | Plot height |
| `--dpi N` | 300 | Resolution |
| `--no-background` | off | Remove chromosome bands |

---

## Mode 3 — R API

### GWAS

```r
library(RasterMan)

# Read data (supports .gz, auto-detects separator)
gwas <- read_gwas("results.mlma",
                  chr_col = "Chr", bp_col = "bp", p_col = "p")

# Prepare cumulative positions
prep     <- prep_gwas(gwas)
gwas_df  <- prep$gwas
chr_info <- prep$chr_info

# Plot parameters
params <- list(
  sig_threshold     = 5e-8,
  color_scheme      = "modern",      # modern/classic/colorblind/pastel/vibrant/gray
  # color_even      = "#2E86AB",     # override scheme color for even chromosomes
  # color_odd       = "#A23B72",     # override scheme color for odd chromosomes
  anno_snp          = TRUE,
  window_size       = 500000,
  max_anno          = 50,
  anno_gene         = FALSE,
  gene_distance     = 50000,
  raster_method     = "auto",
  output_format     = "png",
  mode              = "manhattan",
  adaptive_sampling = TRUE,
  anno_threshold    = 5e-8
)

p  <- plot_gwas(gwas_df, chr_info, params)
pq <- plot_qq(gwas_df, params)

ggplot2::ggsave("manhattan.png", p,  width = 14, height = 6, dpi = 300)
ggplot2::ggsave("qqplot.png",    pq, width = 6,  height = 6, dpi = 300)
```

---

### Cis-eQTL

```r
library(RasterMan)

# Read data
cis <- read_cis("cis.txt.gz",
                chr_col  = "Chr",
                bp_col   = "Pos",
                p_col    = "P",
                gene_col = "gene_name")

# Prepare
prep     <- prep_cis(cis)
cis_df   <- prep$gwas
chr_info <- prep$chr_info

# Lead SNPs
lead <- lead_snps(cis_df, min_gene_logp = 5)

# Highlight genes (optional: vector of gene names, or NULL)
highlight <- readLines("sig_genes.txt")

# Plot parameters
params <- list(
  sig_threshold   = 1e-5,
  color_scheme    = "colorblind",
  # color_even    = "#0173B2",
  # color_odd     = "#DE8F05",
  highlight_color = "#E74C3C",
  highlight_size  = 1.5,
  point_size      = 0.8,
  label_size      = 3,
  max_labels      = 50,
  chr_background  = TRUE,
  raster_method   = "auto",
  raster_width    = 2000L,
  raster_height   = 1000L,
  force_raster    = FALSE,
  no_raster       = FALSE
)

p <- plot_cis(cis_df, chr_info, params, lead, highlight)
ggplot2::ggsave("cis_manhattan.png", p, width = 14, height = 7, dpi = 300)
```

---

### Trans-eQTL

```r
library(RasterMan)

# Read and prepare data
prep <- prep_trans(
  read_trans("trans.txt.gz"),
  chr_col        = "Chr",
  pos_col        = "Pos",
  p_col          = "P",
  gene_chr_col   = "gene_chr",
  gene_start_col = "gene_start",
  gene_end_col   = "gene_end",
  slope_col      = "beta"          # NULL → colour by -log10(P) instead
)

# Plot parameters
params <- list(
  color_scheme  = "modern",        # controls gradient low/high
  # color_low   = "#2E86AB",       # override manually
  # color_high  = "#A23B72",
  raster_method = "auto",
  force_raster  = FALSE,
  no_raster     = FALSE,
  raster_width  = 2000L,
  title         = "Trans-eQTL Manhattan"
)

p <- plot_trans(prep, prep$snp_chr_info, prep$gene_chr_info, params)
ggplot2::ggsave("trans_manhattan.png", p, width = 10, height = 10, dpi = 300)
```

---

## Color Schemes

GWAS and cis-eQTL plots use **alternating odd/even chromosome colors**. Trans-eQTL uses a **diverging gradient** for effect direction (negative → low color, positive → high color).

| Scheme | Odd chr / Negative effect | Even chr / Positive effect |
|--------|--------------------------|---------------------------|
| `modern` | `#A23B72` | `#2E86AB` |
| `classic` | `#FFA500` | `#4169E1` |
| `colorblind` | `#DE8F05` | `#0173B2` |
| `pastel` | `#FFB347` | `#AEC6CF` |
| `vibrant` | `#FF6B6B` | `#4ECDC4` |
| `gray` | `#D3D3D3` | `#808080` |

---

## Rasterization Methods

| Method | Recommended for | Notes |
|--------|-----------------|-------|
| `auto` | All datasets | Selects best method automatically |
| `smart` | 100k – 2M pts | Density-aware; preserves significant signals |
| `multilevel` | 2M – 30M pts | Fine grid for signals + coarse background ★ |
| `adaptive` | 500k – 5M pts | Subdivides high-density regions |
| `basic` | Any size | Fastest; may obscure weak signals |
| `off` | < 200k pts | Standard ggplot2 vector rendering |

**Performance reference (cis-eQTL, 31M data points):**

| Method | Time | Quality | Memory |
|--------|------|---------|--------|
| `off` | ~30 min | 100% | overflow |
| `basic` | ~2 min | 85% | low |
| `smart` | ~1.5 min | 92% | medium |
| `multilevel` | ~1 min | 95% | medium ★ |
| `adaptive` | ~2.5 min | 93% | high |

---

## Input Format Reference

| Software | `--chr-col` | `--bp-col` | `--p-col` | `--snp-col` |
|----------|------------|-----------|---------|-----------|
| GCTA `.mlma` | `Chr` | `bp` | `p` | `SNP` |
| PLINK `.assoc` | `CHR` | `BP` | `P` | `SNP` |
| BOLT-LMM | `SNP_CHR` | `SNP_BP` | `P_BOLT_LMM` | `SNP` |
| GEMMA | `chr` | `ps` | `p_wald` | `rs` |
| SAIGE | `CHR` | `POS` | `p.value` | `SNPID` |
| REGENIE | `CHROM` | `GENPOS` | `LOG10P`* | `ID` |

\* REGENIE outputs −log₁₀(P); `read_gwas()` auto-detects and converts.

---

## License

GPL (≥ 3) © 2026 Wentao Cai · [IASBreeding](https://iasbreeding.cn)
