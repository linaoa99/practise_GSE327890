# Practise_GSE327890
Here, you can find me, Lina, practising running the [nf-core/rnaseq](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE327890) pipeline on the open-source available GEO dataset with the accession number: [GSE327890](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE327890). The *end goal* of practising this pipeline is to apply it with a similar setup on experimental data generated in our own lab.

## Legend
1. GEO Dataset Information
2. Directory and File Information
3. Pipeline Overview


## 1. GEO Dataset Information

**Contributors:** Tsubaki T, Chijimatsu R, Saito T

**Title:** RNA-seq of human synovium from patients with osteoarthritis, rheumatoid arthritis, anterior cruciate ligament rupture, and meniscus tear

**Organism:** Homo sapiens

**Experiment type:** Expression profiling by high throughput sequencing

**Summary:** RNA sequencing (RNA-seq) was performed on human synovial tissues obtained from patients with osteoarthritis (OA), rheumatoid arthritis (RA), anterior cruciate ligament rupture (ACLR), and meniscus tear (MT) to characterize transcriptional programs associated with synovial pathology. Osteoarthritis is a heterogeneous joint disease exhibiting substantial variability in synovial inflammation, vascularization, and fibrotic remodeling. In this study, synovial tissue samples were collected from patients undergoing joint surgery, and bulk RNA-seq was conducted to profile gene expression across different disease contexts. These transcriptomic data were generated as part of a broader effort to investigate the molecular basis of synovial tissue heterogeneity and its relationship to histological architecture and disease severity.

**Overall design:**	Human synovial tissue samples were obtained from patients with OA, RA, ACLR, and MT undergoing surgery or biopsy at the University of Tokyo Hospital with informed consent and institutional ethics approval. Total RNA was extracted from synovial tissues, and bulk RNA sequencing libraries were prepared using the NEBNext Ultra II RNA Library Prep Kit for Illumina and sequenced on an Illumina NovaSeq 6000 platform. Raw reads were quality-controlled using fastp, aligned to the GRCh38 reference genome using STAR, and gene expression levels were quantified with RSEM.

## 2. Directory and File Overview

- `.devcontainer/devcontainer.json`: container definition
- `.nextflow`: contains cache database, run history and potential plugins. It is necessary for `-resume` of the pipe
- `pixi.toml` / `pixi.lock`: tool versions
- `.gitignore`: excludes `.pixi/`, `.nextflow*`, `.nextflow-home/`, `work/`, `results/`

## 3. Pipeline Overview

### Pipelines

| Step | Pipeline | Purpose |
|------|----------|---------|
| 1 | [nf-core/fetchngs](https://nf-co.re/fetchngs) | Downloads the FASTQ files and metadata for the GEO/SRA accessions and creates a samplesheet |
| 2 | [nf-core/rnaseq](https://nf-co.re/rnaseq) | QC, alignment/quantification (STAR, Salmon), StringTie, DESeq2 QC and a MultiQC report |


Pipeline versions: fetchngs `<version>`, rnaseq `<version>` (pinned with `-r`).

<img width="2028" height="708" alt="image" src="https://github.com/user-attachments/assets/2af18808-e077-43a3-a170-96e17388f8c0" />

### Environment

Everything runs in a reproducible **VS Code Dev Container** on Windows with WSL2:

- **Host:** Windows + WSL2 (Ubuntu) + Docker Desktop
- **Container:** `mcr.microsoft.com/devcontainers/base:ubuntu`, defined in `.devcontainer/devcontainer.json`
- **Docker:** the container uses the host's Docker Desktop (`docker-outside-of-docker`),
  so Nextflow can start the pipelines' tool containers (`-profile docker`)
- **Same-path mount:** the project is mounted at the same path inside the container as in
  WSL, so the host Docker daemon can see the files Nextflow passes to the tool containers
- **Pixi** (`pixi.toml`) manages the tools: Python 3.12, Nextflow (>= 25.10), nf-core tools
  and OpenJDK 21
- `NXF_HOME` points to `.nextflow-home/` inside the project, so pipeline scripts are
  visible to the host Docker daemon

The project must be stored in the WSL filesystem (`~/projects/...`), not on a Windows drive.

### Usage

1. Open the project from the Ubuntu (WSL) terminal: `code .`
2. Run **Dev Containers: Reopen in Container**
3. Check the setup:
```bash
   pixi run nextflow -version
   docker run --rm -v "$PWD":/data ubuntu:22.04 ls /data   # should list pixi.toml
```
4. Test run on the built-in test data:
```bash
   pixi run nextflow run nf-core/rnaseq -r <version> -profile test,docker --outdir results
```
5. Download the GEO data:
```bash
   pixi run nextflow run nf-core/fetchngs -r <version> -profile docker \
       --input ids.csv --outdir data
```
6. Run rnaseq on the real data:
```bash
   pixi run nextflow run nf-core/rnaseq -r <version> -profile docker \
       --input data/samplesheet/samplesheet.csv --genome <genome> --outdir results
```

Use `-resume` to continue an interrupted run.

### Notes

- Java is pinned to 21 because a newer Java caused a `process hasn't exited` error in Nextflow.
- Run all commands via `pixi run ...` or inside `pixi shell`.



