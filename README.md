<div align="center">

<img src="https://img.shields.io/badge/Nextflow-≥23.10-brightgreen?style=for-the-badge&logo=nextflow&logoColor=white" />
<img src="https://img.shields.io/badge/DSL2-Modular-jade?style=for-the-badge" />
<img src="https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
<img src="https://img.shields.io/badge/Singularity-HPC%20Ready-orange?style=for-the-badge" />
<img src="https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge" />
<img src="https://img.shields.io/badge/CI-passing-success?style=for-the-badge" />

<br/><br/>

# 🧬 BacTrack-NF

### Clinical Bacterial Whole-Genome Sequencing Pipeline

*From raw reads to audit-ready report — in one command.*

**[📄 Pipeline Docs](https://purva-21.github.io/BacTrack-NF/)** &nbsp;·&nbsp;
**[📊 Example Report](https://purva-21.github.io/BacTrack-NF/report.html)** &nbsp;·&nbsp;
**[🐛 Issues](https://github.com/Purva-21/BacTrack-NF/issues)**

</div>

---

## 🔬 What is BacTrack-NF?

**BacTrack-NF** is a fully automated, reproducible bioinformatics pipeline built for clinical and research microbiology labs that need to go from raw Illumina sequencing reads to a clinically meaningful report — without stitching together scripts that break.

It wraps six core analysis modules into a single Nextflow DSL2 workflow, containerised with Docker and Singularity, and produces a structured HTML + JSON report designed for clinical record-keeping and infection control.

> Built by a PhD researcher in clinical metagenomics. Designed to be handed to a lab technician with one command.

---

## 💡 Why Was BacTrack-NF Built?

Whole-genome sequencing is now standard in clinical microbiology. But the bioinformatics hasn't kept up.

```
❌  The status quo in most clinical labs
────────────────────────────────────────────────────────────────
  • Ad-hoc bash scripts stitched together per-sample
  • Tool version mismatches between runs → non-reproducible results
  • No structured output → results buried in TSV files
  • Hard to deploy on air-gapped hospital HPCs
  • No outbreak alerting — cluster detection is manual
  • Cannot be audited → fails clinical governance requirements
```

```
✅  What BacTrack-NF solves
────────────────────────────────────────────────────────────────
  • Single Nextflow DSL2 command → fully reproducible across runs
  • Every tool pinned to a versioned Docker/Singularity container
  • Structured HTML + JSON report → drop into clinical records
  • Singularity profile → works on air-gapped hospital HPCs
  • SNP-distance outbreak alerting → configurable cluster thresholds
  • Full audit trail → timestamps, tool versions, parameters logged
```

Existing tools like **Bactopia** and **Nullarbor** are research-focused. BacTrack-NF is built specifically for **clinical-grade reproducibility**.

---

## 🏗️ Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          BacTrack-NF  v1.0.0                           │
│                    Clinical Bacterial WGS Pipeline                      │
└─────────────────────────────────────────────────────────────────────────┘

  samplesheet.csv
       │
       ▼
  ┌─────────┐     ┌──────────┐     ┌────────────┐     ┌─────────────┐
  │  01 QC  │────▶│ 02 Assem.│────▶│ 03 Annotate│────▶│ 04 AMR/VF  │
  │         │     │          │     │            │     │             │
  │ FastQC  │     │ Shovill  │     │  Prokka    │     │ AMRFinder+  │
  │  fastp  │     │  SPAdes  │     │   QUAST    │     │  RESFinder  │
  └─────────┘     └──────────┘     └────────────┘     └─────────────┘
                                                              │
                                                              ▼
                                                    ┌─────────────────┐
                                                    │  05 MLST Typing │
                                                    │                 │
                                                    │    mlst (tseemann)│
                                                    │    Kleborate    │
                                                    └────────┬────────┘
                                                             │
                                                             ▼
                                                    ┌─────────────────┐
                                                    │  06 Phylogeny   │
                                                    │                 │
                                                    │    Snippy       │
                                                    │   FastTree2     │
                                                    └────────┬────────┘
                                                             │
                                                             ▼
                                             ┌───────────────────────────┐
                                             │   report.html + report.json│
                                             │   Audit-ready · Timestamped│
                                             │   QC flags · AMR profiles  │
                                             │   SNP cluster alerts       │
                                             └───────────────────────────┘
```

Each module is **independently testable**, containerised, and skippable via `--skip_*` flags.

---

## ⚙️ Module Overview

| # | Module | Tools | Output |
|---|--------|-------|--------|
| 01 | **Input QC** | FastQC · fastp | Trimmed reads · MultiQC report |
| 02 | **Assembly** | Shovill · SPAdes | Contigs FASTA · assembly stats |
| 03 | **Annotation** | Prokka · QUAST | GFF · GBK · N50 · coverage |
| 04 | **AMR Detection** | AMRFinder+ · RESFinder | Resistance gene table · drug classes |
| 05 | **MLST Typing** | mlst · Kleborate | Sequence type · allele profiles |
| 06 | **Phylogeny** | Snippy · FastTree2 | SNP matrix · Newick tree · cluster alerts |

---

## 🚀 Quick Start

**Requirements:** Java 17+ · Docker or Singularity · Nextflow ≥ 23.10

```bash
# 1 — Install Nextflow
curl -s https://get.nextflow.io | bash

# 2 — Run BacTrack-NF with Docker
nextflow run Purva-21/BacTrack-NF \
  --input    samplesheet.csv    \
  --outdir   ./results          \
  --genome   GCF_000240185.1    \
  --snp_threshold 10            \
  -profile   docker

# 3 — Open the report
open results/report.html
```

**Samplesheet format (`samplesheet.csv`):**
```csv
sample,fastq_1,fastq_2
KP-001,/data/KP001_R1.fastq.gz,/data/KP001_R2.fastq.gz
KP-002,/data/KP002_R1.fastq.gz,/data/KP002_R2.fastq.gz
```

---

## 🖥️ Deployment Profiles

```
┌──────────────────┬────────────────────────────────────────────────┐
│ Profile          │ Command                                        │
├──────────────────┼────────────────────────────────────────────────┤
│ Local + Docker   │ -profile docker                                │
│ HPC + Singularity│ -profile slurm,singularity                     │
│ AWS Batch        │ -profile aws                                   │
│ Google Cloud     │ -profile gcp                                   │
└──────────────────┴────────────────────────────────────────────────┘
```

**For air-gapped hospital HPC (Slurm + Singularity):**
```bash
export NXF_SINGULARITY_CACHEDIR=/path/to/shared/cache

nextflow run Purva-21/BacTrack-NF \
  --input  samplesheet.csv              \
  --outdir /scratch/$USER/bactrack_out  \
  -profile slurm,singularity            \
  -resume
```

---

## 🛡️ Key Features

<table>
<tr>
<td width="50%">

### 📋 Audit-Ready Reports
Every run produces a timestamped HTML + JSON report with:
- Per-sample QC flags (pass / warn / fail)
- AMR gene table with drug class and % identity
- Chain-of-custody metadata (tool versions, params, timestamps)
- Exportable for clinical record-keeping systems

</td>
<td width="50%">

### 🚨 Outbreak Cluster Alerting
- SNP-distance matrix computed from core-genome alignment
- Configurable cluster threshold: `--snp_threshold N`
- Isolates within N SNPs automatically flagged in the report
- Critical for real-time infection control decisions

</td>
</tr>
<tr>
<td width="50%">

### 🔒 Air-Gap & HPC Ready
- All containers versioned and pre-pullable
- Singularity profile for no-internet hospital HPCs
- Zero dependency conflicts — everything inside containers
- `NXF_SINGULARITY_CACHEDIR` for shared HPC cache

</td>
<td width="50%">

### 🧪 CI-Tested with nf-test
- Each module ships with nf-test unit tests
- Snapshot testing on every commit via GitHub Actions
- Ensures outputs remain consistent across tool version bumps
- Test data included for all six modules

</td>
</tr>
</table>

---

## 🔧 Configuration

```groovy
// nextflow.config — key parameters
params {
  input           = null          // path to samplesheet.csv
  outdir          = "results"     // output directory
  genome          = null          // reference genome accession
  snp_threshold   = 20            // SNP cluster alert distance
  min_coverage    = 30            // minimum depth to pass QC
  skip_annotation = false         // skip Prokka step
  skip_phylogeny  = false         // skip Snippy/FastTree step
}

// Resource tuning per process
process {
  withName: 'SHOVILL' { cpus = 8;  memory = '32.GB'; time = '2.h' }
  withName: 'FASTTREE' { cpus = 16; memory = '16.GB' }
}
```

---

## 📁 Output Structure

```
results/
├── multiqc/
│   └── multiqc_report.html        # aggregated QC across all samples
├── assembly/
│   └── {sample}/
│       ├── contigs.fasta
│       └── quast_report/
├── annotation/
│   └── {sample}/
│       ├── sample.gff
│       └── sample.gbk
├── amr/
│   └── {sample}/
│       └── amrfinder_results.tsv
├── mlst/
│   └── mlst_summary.tsv
├── phylogeny/
│   ├── core.snp.newick
│   └── snp_distance_matrix.tsv
└── report.html                    ← audit-ready clinical report
    report.json
```

---

## 📚 Included Tools & Citations

| Tool | Version | Citation |
|------|---------|----------|
| FastQC | 0.12.1 | Andrews S. (2010) |
| fastp | 0.23.4 | Chen et al. (2018) *Bioinformatics* |
| Shovill | 1.1.0 | Seemann T. (2018) |
| SPAdes | 3.15.5 | Bankevich et al. (2012) *J Comp Bio* |
| Prokka | 1.14.6 | Seemann T. (2014) *Bioinformatics* |
| QUAST | 5.2.0 | Gurevich et al. (2013) *Bioinformatics* |
| AMRFinder+ | 4.0.23 | Feldgarden et al. (2021) *Sci Rep* |
| mlst | 2.23.0 | Seemann T. (2016) |
| Kleborate | 2.3.2 | Lam et al. (2021) *Nat Commun* |
| Snippy | 4.6.0 | Seemann T. (2015) |
| FastTree2 | 2.1.11 | Price et al. (2010) *PLoS ONE* |

---

## 🤝 Contributing

Contributions welcome. Please open an issue first for major changes.

```bash
git clone https://github.com/Purva-21/BacTrack-NF.git
cd BacTrack-NF
# install nf-test, then:
nf-test test tests/
```

---

## 👩‍🔬 About

Built by **Purva Gohil** — PhD researcher in clinical metagenomics, working at the intersection of clinical microbiology and computational genomics.

BacTrack-NF grew out of frustration with fragmented tooling in a clinical WGS setting. The goal: something reproducible, audit-ready, and simple enough to hand to a lab technician with one command.

**Skills:** Python · R/Bioconductor · Bash · Nextflow DSL2 · Snakemake · Docker · Singularity · QIIME2 · scikit-learn · GitHub Actions

📧 [purvagohil1706@gmail.com](mailto:purvagohil1706@gmail.com) &nbsp;·&nbsp; 🔗 [LinkedIn](https://linkedin.com/in/purva-gohil)

---

<div align="center">

**MIT License · © 2026 Purva Gohil**

*If you use BacTrack-NF in your research, please cite the tools listed above.*

</div>
