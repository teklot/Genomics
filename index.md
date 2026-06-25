# Year 1 Genomics Study Roadmap

**Estimated total time: 400–600 hours** (8–10 hrs/week over 50 weeks, with 2 weeks buffer)

---

## How to Read This Roadmap

- Each module lists **prerequisites** — skip these if you're solid on the topic
- **Estimated hours** include reading, exercises, and review
- **Exercises** are required — they're the difference between "knowing about" and "being able to"
- The final portfolio project accounts for the largest single time block

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
| 06 | [`06-sequencing-technology.md`](06-sequencing-technology.md) | 01–05 | 10–14 |
| 07 | [`07-read-lengths-coverage.md`](07-read-lengths-coverage.md) | 01–05 | 8–12 |
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
| 11 | [`11-fasta-fastq.md`](11-fasta-fastq.md) | 06–10 | 10–14 |
| 12 | [`12-sam-bam-cram.md`](12-sam-bam-cram.md) | 06–11 | 12–16 |
| 13 | [`13-vcf-bcf.md`](13-vcf-bcf.md) | 06–11 | 10–14 |
| 14 | [`14-samtools-deep-dive.md`](14-samtools-deep-dive.md) | 12 | 12–16 |
| 15 | [`15-bcftools-deep-dive.md`](15-bcftools-deep-dive.md) | 13–14 | 10–14 |

**Phase total: 54–74 hours**

**Milestone check:** Can you pipe `samtools view` + `grep` + `awk` to answer a question about reads in a BAM file? Can you write a Python script that parses a VCF and computes Ti/Tv ratio?

---

## Phase 4 — Workflows & Cloud (Weeks 13–16)

**Goal:** Automate pipelines at scale using Nextflow, containers, and AWS.

| # | File | Prerequisites | Est. Hours |
|---|------|---------------|------------|
| 16 | [`16-nextflow-dsl2.md`](16-nextflow-dsl2.md) | 10–15, Linux | 16–24 |
| 17 | [`17-containers-bioinformatics.md`](17-containers-bioinformatics.md) | Linux, Docker basics | 10–14 |
| 18 | [`18-aws-genomics.md`](18-aws-genomics.md) | Linux, basic networking | 14–20 |
| 19 | [`19-pipeline-integration.md`](19-pipeline-integration.md) | 16–18 | 12–18 |

**Phase total: 52–76 hours**

**Milestone check:** Can you write a Nextflow workflow that runs FastQC → BWA → SAMtools → BCFtools? Can you deploy it on AWS Batch?

---

## Phase 5 — Portfolio Project & Career (Weeks 17–20)

**Goal:** Build and deploy a portfolio project that demonstrates all prior skills. Prepare for interviews.

| # | File | Prerequisites | Est. Hours |
|---|------|---------------|------------|
| 20 | [`20-portfolio-career.md`](20-portfolio-career.md) | All prior | 60–100 |

**Phase total: 60–100 hours**

**Milestone check:** Do you have a public GitHub repo with a working ASP.NET Core + HTMX + Bootstrap dashboard that shows real genomics pipeline results? Can you walk an interviewer through every layer — DNA → base calls → alignment → variants → visualization?

---

## Total Estimated Time

| Phase | Low | High |
|-------|-----|------|
| 1 — Molecular Biology | 42 | 62 |
| 2 — Sequencing & Alignment | 46 | 68 |
| 3 — File Formats & CLI Tools | 54 | 74 |
| 4 — Workflows & Cloud | 52 | 76 |
| 5 — Portfolio & Career | 60 | 100 |
| **Total** | **254** | **380** |

Add 20% buffer for review, setup, troubleshooting: **~300–450 hours**.

---

## Weekly Schedule (Example)

| Day | Activity | Time |
|-----|----------|------|
| Mon | Read & take notes on one section | 1.5 hr |
| Wed | Do the exercises for that section | 1.5 hr |
| Fri | Review prior week, write a summary | 1 hr |
| Sat | Deep work — work on the portfolio project | 3–4 hr |

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
| Nextflow | Seqera's Nextflow training (free, 4-part course) |
