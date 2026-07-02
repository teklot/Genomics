# 06 — Sequencing Technology: How We Read DNA

**Prerequisites:** `01-dna.md`, `02-rna.md`, `03-genes-chromosomes.md`

**See also:** `07-read-lengths-coverage.md`, `08-alignment-theory.md`, `09-quality-scores.md`

---

## The Big Idea

A DNA sequencer is a **physical-to-digital converter**. It takes physical DNA molecules and produces strings of A, C, T, G with per-base quality scores.

```
Physical DNA  →  Sequencer  →  Digital Sequence (FASTQ)
```

---

## 1. Three Generations of Sequencing

| Generation | Technology | Read Length | Throughput | Error Rate |
|-----------|------------|-------------|------------|------------|
| **1st** | Sanger | ~800 bp | Low (96 samples/run) | ~0.001% |
| **2nd** | Illumina (short-read) | 150–300 bp | Very high (billions of reads) | ~0.1–1% |
| **3rd** | PacBio / Nanopore (long-read) | 10,000–100,000+ bp | Moderate | ~1–15% |

---

## 2. Illumina Sequencing-by-Synthesis (The Industry Standard)

Illumina dominates the market (~80%+ of all sequencing). Understanding its quirks is essential.

```
Three Stages: [1. Library Prep] → [2. Cluster Gen.] → [3. SBS Sequencing]
                Fragment DNA     Bridge Amplification   Massively Parallel
                + Adaptors

Inside One Sequencing Cycle (×150 cycles):
  [Add Nucleotide] → [Wash] → [Image] → [Cleave]
  (Fluorescent +     (Remove   (Laser    (Ready for
   blocked dNTPs)     unbound)  excite)    next cycle)
         ↓
  Repeat for each base position

Flow Cell: [Lane 1] [Lane 2] [Lane 3] [Lane 4] ...
           (Millions of clusters per lane)
```

### How It Works

```
Step 1: Library Preparation
  DNA → fragment → add adapters → attach to flow cell

Step 2: Cluster Generation
  Bridge amplification creates ~1000 identical copies of each fragment
  (called a "cluster")

Step 3: Sequencing-by-Synthesis
  - Fluorescently labeled nucleotides are added one at a time
  - Each base emits a specific color when incorporated
  - A camera captures the color for every cluster simultaneously
  - The cycle repeats for each base position

Step 4: Base Calling
  - Images → intensity values → base calls + quality scores
  - Output: FASTQ (sequences + quality scores)
```

### Illumina Platform Comparison

| Platform | Max Reads | Read Length | Run Time | Best For |
|----------|-----------|-------------|----------|----------|
| MiniSeq | 25M | 2×150 bp | 24 hr | Small projects, QC |
| MiSeq | 25M | 2×300 bp | 55 hr | Targeted/amplicon |
| NextSeq 550 | 400M | 2×150 bp | 29 hr | Exomes, RNA-seq |
| NovaSeq 6000 | 10B | 2×150 bp | 44 hr | WGS, population |
| NovaSeq X | 26B | 2×150 bp | 48 hr | Ultra-high throughput |

### Key Concepts for Illumina

```
  [Adapter]─┤                                     ├─[Adapter]
            │  Read 1 (150 bp) →                  │
            │                    ← Read 2 (150 bp) │
            └─────────────────────────────────────┘
            ◄───── Insert Size (350 bp) ──────────→
            ◄─ Gap (50 bp) ─→
            Gap = Insert - (R1 + R2)
```

**Paired-end sequencing** reads both ends of each fragment:

```
Fragment:  ┌──────────────────────────────────────┐
           │ 150 bp              150 bp           │
Read 1:    └────────────────────→                 │
Read 2:    ◄────────────────────┘                 │
           └──────────────────────────────────────┘
                Insert size (variable)
```

**Insert size** = the distance between the two reads. For a 350 bp insert with 2×150 reads, the gap between reads is 50 bp.

### Illumina Error Profile

- **Quality decreases** toward the end of reads
- GGC motifs tend to be error-prone
- Substitutions dominate (not indels)
- The first ~5 bases are often low quality (template switching noise)

---

## 3. Oxford Nanopore (Long-Read)

Nanopore sequences single molecules in real time.

### How It Works

```
- A protein nanopore is embedded in an electrically-resistant membrane
- A single DNA strand passes through the pore
- Each base causes a characteristic change in ionic current
- The current signal is decoded into bases in real time
```

```
  cis side (Input DNA)
  ┌─────────────────────────────────┐
  │   A ─ T ─ G ─ C ─ A ─ T ─ G    │
  │              ↓                  │
  │  ┌─────── Nanopore ───────┐     │
  ├──│  Biological nanopore   │─────┤ ← Lipid Bilayer Membrane
  │  │  embedded in membrane  │     │
  │  └────────────────────────┘     │
  │              ↓                  │
  │   Ionic Current (pA) ─~‾~‾~‾   │
  │   Unique disruption per base    │
  │              ↓                  │
  │   Base Caller (Neural Network)  │
  │   Real-time base calling        │
  └─────────────────────────────────┘
  trans side (Output DNA)

Key: Real-time, ultra-long reads, portable (MinION)
```

### Key Characteristics

```
Read length: Limited only by DNA preparation (50,000–2,000,000+ bp)
Throughput:  ~20–100 Gb per flow cell (depends on mode)
Error rate:  ~5–15% (mostly indels in homopolymer runs)
Base calling: Done in real time by neural networks (Guppy, Dorado)
```

### Why Long Reads Matter

- **Resolve repetitive regions** (short reads can't span them)
- **Detect structural variants** (deletions, insertions, inversions)
- **Phasing** (determine which parent a variant came from)
- **De novo assembly** (build genomes without a reference)

```
Short reads:   ACTGACTGACTGACTG  ← repetitive, ambiguous
Long read:     ACTGACTGACTGACTG-NANOPORE-GCATGCATGCATGCAT
                                                    ↑ unambiguous
```

### Nanopore Error Profile

- Homopolymer errors (AAAA vs AAA vs AAAAA) — hardest problem
- Base-calling errors are correlated (not random)
- Consensus accuracy can reach Q50 (99.999%) with enough coverage

---

## 4. PacBio HiFi (Long-Read, High Accuracy)

PacBio HiFi reads are the "best of both worlds": long reads with Illumina-quality accuracy.

### How It Works

```
- Circular sequencing: the same DNA molecule is read multiple times
- A "circular consensus sequence" (CCS) is derived from multiple passes
- Result: 10–25 kb reads with >99.9% accuracy (Q30+)
```

**Analogy:** Running the same test 10 times and taking the majority vote.

```
Pass 1:  ACTGGCTAA  ← errors
Pass 2:  ACTGGCTAA
Pass 3:  ACTGGCTAA  ← accurate consensus
...
CCS:     ACTTGCTAA  ← final, high-quality
```

### Comparison

| | Illumina | Nanopore | PacBio HiFi |
|---|----------|----------|-------------|
| Read length | 150 bp | 50 kb+ | 15–25 kb |
| Accuracy | Q30+ (99.9%) | Q10–Q20 (90–99%) | Q30+ (99.9%) |
| Single molecule | No (cluster) | Yes | Yes (cyclic) |
| Capital cost | $$ | $ | $$$ |
| Run cost/Gb | $ | $$ | $$$ |

---

## 5. The Sequencing Pipeline (High Level)

```
Sample → DNA extraction → Library prep → Sequencer → Base calling → FASTQ
```

### Library Preparation

This is the most labor-intensive step and where many pipelines fail:

```
DNA → fragment (sonication/enzymatic) → end repair → A-tailing →
adapter ligation → size selection → PCR amplification → QC
```

- **DNA quality** and **quantity** must meet thresholds
- **PCR duplicates** are artifacts — multiple reads from the same original molecule
- **Adapter contamination** must be trimmed (see `10-quality-control.md`)

### Multiplexing

Multiple samples can be run in one lane by attaching unique **barcodes** (indexes):

```
Sample 1: [Index_A] ACTG... [Index_A]
Sample 2: [Index_B] ACTG... [Index_B]

Demultiplexing: Split FASTQ by index sequence → one FASTQ per sample
```

---

## 6. Practical Considerations for Platform Engineers

### Storage Requirements

| Data Type | Size |
|-----------|------|
| 30× human WGS (FASTQ) | ~200–300 GB |
| 30× human WGS (BAM) | ~60–100 GB |
| 30× human WGS (CRAM) | ~20–40 GB |
| 1 million reads (FASTQ) | ~250 MB |
| Single flow cell (NovaSeq S4) | ~6 TB (FASTQ) |

### Compute Requirements

```
30× WGS:
  Alignment:  ~10–20 CPU-hours
  Variant calling: ~20–40 CPU-hours
  Total: ~100 GB RAM peak (some tools)
```

### Cost Examples

```
Whole genome (30× Illumina):     ~$600–$1,000
Whole exome (100× Illumina):     ~$100–$300
RNA-seq (50M reads):              ~$50–$200
Nanopore whole genome:            ~$1,000–$2,000
PacBio HiFi whole genome:         ~$2,000–$5,000
```

---

## Exercises

1. If a NovaSeq S4 flow cell produces 10 billion reads at 2×150 bp, how many gigabases is that? How many 30× human genomes does that cover?
2. Compare the throughput, read length, and error profiles of Illumina, PacBio, and Nanopore sequencing. Which technology would you choose for: (a) detecting novel splice junctions, (b) assembling a novel genome, (c) finding rare somatic variants?
3. Why do Nanopore reads have more indels while Illumina has more substitutions?
4. Calculate: A sequencing lab charges $800 per 30× WGS sample. If a sequencer costs $1M and needs 1 FTE operator at $80k/yr, how many samples must the lab run per year to break even? (Assume $200/sample consumable cost.)

---

## Key Terms

| Term | Definition |
|------|------------|
| Sequencing-by-synthesis (SBS) | Illumina's method — add bases, capture fluorescence, repeat |
| Paired-end | Sequencing both ends of a DNA fragment |
| Flow cell | The consumable surface where sequencing occurs |
| Cluster | ~1000 identical copies of one DNA fragment (Illumina) |
| Base calling | Converting raw instrument signals to bases + qualities |
| Demultiplexing | Splitting pooled samples by barcode |
| Library prep | Processing DNA into a sequencer-ready format |
| PCR duplicate | Sequencing artifact — same molecule read twice |
| Adapter | Synthetic DNA sequence attached during library prep |
| Insert size | Distance between paired-end reads on the original fragment |

---

## Next Steps

→ `07-read-lengths-coverage.md` — How coverage affects confidence  
→ `08-alignment-theory.md` — Mapping reads to the reference  
→ `09-quality-scores.md` — The Phred quality score system  

---

*"A sequencer is a camera that takes pictures of chemistry. The base caller is the OCR."*
