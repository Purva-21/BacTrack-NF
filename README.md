# BacTrack-NF

**Clinical Bacterial WGS Pipeline** — Nextflow DSL2 · Docker Ready · v1.0.0-beta

A reproducible, containerized Nextflow pipeline for clinical microbiology labs — automating QC, assembly, AMR typing, MLST, and phylogenetic surveillance into a single audit-ready HTML report.

## Live Demo

- **[Pipeline Documentation](https://purva-21.github.io/BacTrack-NF/)** — pipeline architecture and usage guide
- **[Example Analysis Report](https://purva-21.github.io/BacTrack-NF/report.html)** — real output from `assembly_contigs.fasta`

## Real Data Results (sample1)

Analysis run on `assembly_contigs.fasta` — 36 contigs, ~3.97 Mb assembly, ~192.8× mean coverage.

| Module | Tool | Result |
|--------|------|--------|
| Assembly QC | Custom QC | 36 contigs · N50 = 1.01 Mb · 192.8× · **PASS** |
| Annotation | Prokka 1.14.6 | 3,925 CDS · 74 tRNA · 4 rRNA · 1 tmRNA |
| AMR Screening | AMRFinder+ · Abricate | **5 AMR genes** across 4 drug classes |
| Organism ID | CARD homology | Putative *Bacillus velezensis* |
| MLST | — | No scheme available for this species |

### AMR Genes Detected

| Gene | Drug Class | % Identity | Database |
|------|-----------|-----------|----------|
| clbA | Lincosamide · Macrolide · Streptogramin · Oxazolidinone | 92.55% | AMRFinder+ |
| cfr(B)_3 | Chloramphenicol · Florfenicol · Linezolid · Clindamycin | 88.67% | ResFinder |
| tet(L)_5 | Tetracycline · Doxycycline | 87.29% | ResFinder |
| satA | Streptothricin | 84.39% | AMRFinder+ |
| rphB | Rifamycin (lower confidence) | 80.95% | CARD |

> **Note:** clbA, cfr(B), and satA co-localise on `assembly_contig_8`, suggesting a mobile genetic element.

## Pipeline Architecture

```
Raw Reads → [FastQC / fastp] → [Shovill / SPAdes] → [Prokka / QUAST]
         → [AMRFinder+ / RESFinder] → [mlst / Kleborate] → [Snippy / FastTree2]
         → HTML + JSON Report
```

## Quick Start

```bash
# Install Nextflow (requires Java 17+)
curl -s https://get.nextflow.io | bash

# Run BacTrack-NF
nextflow run Purva-21/BacTrack-NF \
  --input samplesheet.csv \
  --outdir ./results \
  --genome GCF_000240185.1 \
  --snp_threshold 10 \
  -profile docker

# Open report
open results/report.html
```

## Features

- **Audit-Ready HTML Reports** — timestamped HTML + JSON with QC flags, AMR profiles, chain-of-custody metadata
- **Outbreak Cluster Alerting** — SNP-distance matrix with configurable threshold alerts
- **Air-Gap & HPC Ready** — Singularity support for hospital HPCs with no internet access
- **CI-Tested** — each module ships with nf-test unit tests
- **Flexible Config** — pre-built profiles for local, Slurm, AWS Batch, Google Cloud

## Requirements

- Nextflow ≥ 23.10
- Docker or Singularity
- Java 17+

## License

MIT License — © 2026 Purva Gohil
