# Genomics for Software Engineers — Year 1 Curriculum

A progressive, self-contained study guide that builds from molecular biology fundamentals to fluency in genomics file formats and CLI tools.

## Target Audience

- Software engineers transitioning into bioinformatics / platform engineering
- Already comfortable with Linux CLI, at least one programming language (preferably C# or Python)
- Want to reach a **top-tier, high-earning** role combining genomics + cloud + full-stack skills

## What You'll Be Able to Do

- Explain a sequencing pipeline end-to-end
- Read, write, and manipulate FASTQ, BAM, CRAM, and VCF files
- Run quality control with FastQC and fastp
- Align reads with BWA and Minimap2
- Call and filter variants with SAMtools + BCFtools

## Structure

| Phase | Weeks | Topic | Files |
|-------|-------|-------|-------|
| 1 | 1–4 | Molecular Biology for Engineers | `01`–`05` |
| 2 | 5–8 | Sequencing & Alignment | `06`–`10` |
| 3 | 9–12 | File Formats & CLI Tools | `11`–`15` |

**Total: ~142–204 hours core (~170–245 hours with 20% buffer) over 12 weeks (~14–20 hrs/week).**

## How to Use

1. Read files in numerical order — each is standalone but concepts compound.
2. Every file has **exercises** — do them.

## Lessons at a Glance

| # | Lesson | Prerequisites | Description |
|---|--------|---------------|-------------|
| 01 | [DNA: The Source Code](01-dna.md) | None | Double helix, base pairing, antiparallel strands, orientation |
| 02 | [RNA: The Transient Build Artifact](02-rna.md) | 01 | Transcription, splicing, reverse transcription, RNA types |
| 03 | [Genes & Chromosomes](03-genes-chromosomes.md) | 01–02 | Gene structure, chromosome anatomy, genome organization |
| 04 | [Central Dogma](04-central-dogma.md) | 01–03 | DNA→RNA→Protein flow, reading frames, the ribosome |
| 05 | [Mutations & Genetics](05-mutations-genetics.md) | 01–04 | SNVs, indels, structural variants, frameshift effects |
| 06 | [Sequencing Technology](06-sequencing-technology.md) | 01–03 | Illumina SBS, Nanopore, libraries, read types |
| 07 | [Read Lengths & Coverage](07-read-lengths-coverage.md) | 06 | Coverage depth, Poisson statistics, mate pairs |
| 08 | [Alignment Theory](08-alignment-theory.md) | 06–07 | Seed-and-extend, Smith-Waterman, SAM flags, MAPQ |
| 09 | [Quality Scores](09-quality-scores.md) | 06–07 | Phred scores, quality encoding, error probability |
| 10 | [Quality Control](10-quality-control.md) | 06–09 | FastQC, trimming, contamination detection |
| 11 | [FASTA & FASTQ Format](11-fasta-fastq.md) | 06, 09–10 | Record structure, parsing, validation, tools |
| 12 | [SAM/BAM/CRAM Format](12-sam-bam-cram.md) | 08, 11 | 11 required columns, flags, CIGAR, compression |
| 13 | [VCF/BCF Format](13-vcf-bcf.md) | 05, 08, 12 | Header, records, genotypes, INFO/FORMAT fields |
| 14 | [SAMtools Deep Dive](14-samtools-deep-dive.md) | 08, 12 | View, sort, index, flagstat, mpileup |
| 15 | [BCFtools Deep Dive](15-bcftools-deep-dive.md) | 13–14 | Filter, merge, stats, norm, consensus |

## Prerequisites

- Basic Linux command line (`ls`, `cat`, `grep`, `wget`, `tar`, `chmod`)
- Comfort with any programming language (code examples in Python and C#)
- A computer with Docker Desktop or access to a Linux VM
- (Optional) An AWS account with moderate free-tier budget

---

## Phase 1 — Molecular Biology for Engineers (Weeks 1–4)

**Goal:** Speak the biology language — DNA, RNA, genes, proteins, mutations — through software analogies.

| # | File | Prerequisites | Est. Hours |
|---|------|---------------|------------|
| 01 | [`01-dna.md`](01-dna.md) | None | 8–12 |
| 02 | [`02-rna.md`](02-rna.md) | 01 | 6–10 |
| 03 | [`03-genes-chromosomes.md`](03-genes-chromosomes.md) | 01–02 | 8–12 |
| 04 | [`04-central-dogma.md`](04-central-dogma.md) | 01–03 | 10–14 |
| 05 | [`05-mutations-genetics.md`](05-mutations-genetics.md) | 01–04 | 10–14 |

**Phase total: 42–62 hours**

**Milestone check:** Can you explain DNA→RNA→Protein to a fellow engineer using code analogies? Can you write a script that translates a DNA string to a protein sequence?

---

## Phase 2 — Sequencing & Alignment (Weeks 5–8)

**Goal:** Understand how DNA gets turned into digital data and how reads map back to a reference.

| # | File | Prerequisites | Est. Hours |
|---|------|---------------|------------|
| 06 | [`06-sequencing-technology.md`](06-sequencing-technology.md) | 01–03 | 10–14 |
| 07 | [`07-read-lengths-coverage.md`](07-read-lengths-coverage.md) | 06 | 8–12 |
| 08 | [`08-alignment-theory.md`](08-alignment-theory.md) | 06–07 | 14–20 |
| 09 | [`09-quality-scores.md`](09-quality-scores.md) | 06–07 | 6–10 |
| 10 | [`10-quality-control.md`](10-quality-control.md) | 06–09 | 8–12 |

**Phase total: 46–68 hours**

**Milestone check:** Can you explain why 30× coverage is a standard target? Can you interpret a FastQC report and identify a bad lane?

---

## Phase 3 — File Formats & CLI Tools (Weeks 9–12)

**Goal:** Read, write, and manipulate every common genomics file format from the command line and from code.

| # | File | Prerequisites | Est. Hours |
|---|------|---------------|------------|
| 11 | [`11-fasta-fastq.md`](11-fasta-fastq.md) | 06, 09–10 | 10–14 |
| 12 | [`12-sam-bam-cram.md`](12-sam-bam-cram.md) | 08, 11 | 12–16 |
| 13 | [`13-vcf-bcf.md`](13-vcf-bcf.md) | 05, 08, 12 | 10–14 |
| 14 | [`14-samtools-deep-dive.md`](14-samtools-deep-dive.md) | 08, 12 | 12–16 |
| 15 | [`15-bcftools-deep-dive.md`](15-bcftools-deep-dive.md) | 13–14 | 10–14 |

**Phase total: 54–74 hours**

**Milestone check:** Can you pipe `samtools view` + `grep` + `awk` to answer a question about reads in a BAM file? Can you write a Python script that parses a VCF and computes Ti/Tv ratio?

---

## Total Estimated Time

| Phase | Low | High |
|-------|-----|------|
| 1 — Molecular Biology | 42 | 62 |
| 2 — Sequencing & Alignment | 46 | 68 |
| 3 — File Formats & CLI Tools | 54 | 74 |
| **Total** | **142** | **204** |

Add 20% buffer for review, setup, troubleshooting: **~170–245 hours**.

---

## Weekly Schedule (Example)

This curriculum requires **~14–20 hours/week** for a working engineer to finish in 12 weeks.

| Day | Activity | Time |
|-----|----------|------|
| Mon | Read a section, take notes | 1.5 hr |
| Tue | Do exercises and coding problems | 1.5 hr |
| Wed | Continue exercises or deep-dive into examples | 1.5 hr |
| Thu | *(off — life, rest, or catch-up)* | — |
| Fri | Review the week, write a one-page summary | 1 hr |
| Sat | Deep work — main study block | 4–5 hr |
| Sun | Catch-up on missed items or review | 2–3 hr |

**Total: ~14–20 hrs/week** — adjust the weekend blocks up during heavier phases (alignment, file formats).

---

## Quick Reference: When to Use Supplementary Resources

| If you're stuck on… | Try… |
|---------------------|------|
| Biology concepts | YouTube: "DNA replication", "transcription translation" animations |
| Linux / bash | OverTheWire Bandit wargame (first 10 levels) |
| C# / .NET | Microsoft Learn: "Get started with C#" |
| Python | Codecademy or Real Python |
| AWS | AWS Skill Builder — free digital courses for Batch, S3, IAM |
| Docker | Docker's "Get Started" guide |
