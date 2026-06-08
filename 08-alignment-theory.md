# 08 — Alignment Theory: Mapping Reads to the Reference

**Prerequisites:** `06-sequencing-technology.md`, `07-read-lengths-coverage.md`

**See also:** `09-quality-scores.md`, `12-sam-bam-cram.md`

---

## The Big Idea

**Alignment** is the process of figuring out where a sequencing read originated in the genome.

```
Read:    ACTGACTGACTG  →  Where does this go?
                           ↓
Reference: ACTGACTGACTGACTGACTG
                      ↑
                  Right here!
```

**Analogy:** `git diff` against the reference genome — but with errors, gaps, and millions of fragments.

---

## 1. The Alignment Problem

Given:

- A **reference genome** (~3.2 billion bases, 23 chromosomes)
- A **read** (~150 bases for Illumina)

Find:

- The **position** (chromosome + coordinate) where the read fits best
- The **alignment** (which bases match, which differ, which are inserted/deleted)

**Constraints:**

- Reads may have errors (substitutions, indels)
- Reads may span splice junctions (RNA-seq)
- The genome has repetitive regions (ambiguous mapping)
- Speed matters — billions of reads per flow cell

---

## 2. Alignment Algorithms

### Brute Force

```python
# O(N × L) — slide read across every position
def brute_align(read: str, reference: str):
    best_score = -1
    best_pos = -1
    for i in range(len(reference) - len(read)):
        score = sum(1 for a, b in zip(read, reference[i:i+len(read)]) if a == b)
        if score > best_score:
            best_score = score
            best_pos = i
    return best_pos, best_score
```

This is way too slow for a 3 Gb genome. Real aligners use **indexing**.

### FM-Index and Burrows-Wheeler Transform (BWT)

Most modern aligners (BWA, Bowtie2) use the **FM-index** — a compressed full-text index that allows fast substring search.

```
Burrows-Wheeler Transform:
  ACTGACTG$  →  sorted rotations →  last column → index

  FM-index allows O(L) search for exact matches
  (L = read length, not genome size!)
```

**Analogy:** The FM-index is like a **hash map for substrings** — you can find whether a read exists in the genome without scanning the entire genome.

### Seed-and-Extend

Popularized by BLAST, used by Minimap2 and others:

```
Step 1: Find "seeds" — short exact matches between read and reference
Step 2: Extend seeds into full alignments using dynamic programming
Step 3: Choose the best alignment based on score
```

```
Read:    ACTGACTGACTG
         |||   |||          seeds (exact 4-mer matches)
         ↓↓   ↓↓
Ref:     ...ACTG...ACTG...
              ↓ extend
         ACTGACTGACTG  → full alignment
```

### Minimizer Sketching

Used by Minimap2 (hence the name) for long reads:

```
- Pick "minimizers" — the smallest hash value in a sliding window
- Index minimizer positions in the reference
- For each read, find shared minimizers to seed alignment
- Great for noisy long reads (Nanopore, PacBio)
```

---

## 3. Scoring Alignments

Aligners use a **scoring matrix**:

| Event | Score |
|-------|-------|
| Match | +1 |
| Mismatch | -4 |
| Gap open (indel start) | -6 |
| Gap extend (continue indel) | -1 |

The **optimal alignment** maximizes the score.

### The Smith-Waterman Algorithm

The gold standard for local alignment — guarantees the optimal alignment but is O(N × M) in time:

```
     A  C  T  G  A  C  T
  0  0  0  0  0  0  0  0
A 0  1  0  0  0  1  0  0
C 0  0  2  0  0  0  2  0
T 0  0  0  3  0  0  0  3
G 0  0  0  0  4  0  0  0
A 0  1  0  0  0  5  0  0
C 0  0  2  0  0  0  6  0
T 0  0  0  3  0  0  0  7   ← best alignment here
```

Real aligners approximate this with heuristics to run in linear time.

---

## 4. The CIGAR String

The **CIGAR** (Compact Idiosyncratic Gapped Alignment Report) string encodes how a read aligns:

```
Read:    ACTG-ACTGACT
Ref:     ACTGAACTGACT
CIGAR:   4M1D8M

M = Match/mismatch (aligned)
I = Insertion (read has extra bases)
D = Deletion (reference has extra bases)
N = Skipped region (splice junction)
S = Soft-clipped (bases trimmed, still in read)
H = Hard-clipped (bases removed from read)
```

Examples:

```
Read aligns perfectly:       150M
Read with 1 mismatch:        150M (mismatches are "M" too — check NM tag for details)
Read with 2 bp deletion:     50M2D98M
Read with 5 bp insertion:    70M5I75M
Read spanning splice site:   30M50N70M  (RNA-seq exon-exon junction)
Read with adapter trimmed:   5S140M5S  (soft-clipping)
```

```python
def parse_cigar(cigar: str):
    """Parse CIGAR into a list of (length, operation) tuples."""
    import re
    return [(int(m.group(1)), m.group(2)) for m in re.finditer(r'(\d+)([MIDNSHP=X])', cigar)]
```

---

## 5. Alignment Flags (SAM FLAG)

The SAM FLAG field is a **bitwise field** encoding details about the alignment:

```
Dec  Binary    Meaning
───  ──────    ───────
1    1         Read paired
2    10        Read mapped in proper pair
4    100       Read unmapped
8    1000      Mate unmapped
16   10000     Read on reverse strand
32   100000    Mate on reverse strand
64   1000000   First in pair (read 1)
128  10000000  Second in pair (read 2)
256  100000000 Not primary alignment
512  1000000000 Read fails platform/vendor quality checks
1024 10000000000 Read is PCR or optical duplicate
2048 100000000000 Supplementary alignment
```

```python
def explain_flag(flag: int):
    flags = {
        1: "paired",
        2: "proper-pair",
        4: "unmapped",
        8: "mate-unmapped",
        16: "reverse-strand",
        32: "mate-reverse",
        64: "first-in-pair",
        128: "second-in-pair",
        256: "secondary",
        512: "qcfail",
        1024: "duplicate",
        2048: "supplementary",
    }
    return [desc for bit, desc in flags.items() if flag & bit]

# 99 = paired + proper-pair + reverse-strand + first-in-pair
assert explain_flag(99) == ["paired", "proper-pair", "mate-reverse", "first-in-pair"]
```

---

## 6. Popular Aligners

| Aligner | Best For | Method | Index |
|---------|----------|--------|-------|
| **BWA-MEM** | Illumina short reads (70 bp–1 Mb) | BWT + Smith-Waterman | `.bwt`, `.sai` |
| **BWA-MEM2** | Faster BWA-MEM (SIMD optimized) | Same, optimized | `.bwt` |
| **Bowtie2** | Short reads (50–1000 bp) | FM-index + scoring | `.bt2` |
| **Minimap2** | Long reads (PacBio, Nanopore) | Minimizer + chain | `.mmi` |
| **STAR** | RNA-seq (splice-aware) | Suffix array + seed | `.SA` |
| **HISAT2** | RNA-seq (faster, lower memory) | Hierarchical FM-index | `.ht2` |

### How to Run Alignments

```bash
# BWA-MEM alignment
bwa mem -t 8 reference.fasta sample_R1.fastq.gz sample_R2.fastq.gz | samtools sort > aligned.bam

# Minimap2 for long reads
minimap2 -ax map-ont reference.fasta sample.fastq.gz | samtools sort > aligned.bam

# STAR for RNA-seq
STAR --genomeDir star_index/ --readFilesIn sample_R1.fastq.gz sample_R2.fastq.gz --outSAMtype BAM SortedByCoordinate
```

---

## 7. Reference Genomes

An alignment is only as good as the reference:

```bash
# Common reference downloads
wget https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz

# Index for BWA
bwa index hg38.fa

# Index for samtools
samtools faidx hg38.fa
```

### Reference Genome Files

```
hg38.fa            - The genome sequence (FASTA)
hg38.fa.fai        - FASTA index (line offset per chromosome)
hg38.dict          - Sequence dictionary (Picard format)
hg38.bwt           - BWA index (Burrows-Wheeler transform)
```

---

## 8. Why Alignment Matters

```
Raw reads (FASTQ) → Aligned reads (BAM) → Variants (VCF)

Honest alignment:
- Accurate base quality recalibration
- Correctly identified duplicates
- Properly handled indels

Sloppy alignment:
- False positive variants (reads misaligned around indels)
- False negative variants (reads didn't map)
- Biased allele frequency estimates
```

---

## 9. Alignment Statistics

```bash
# Basic alignment stats
samtools flagstat aligned.bam

# Example output:
# 1000000 + 0 in total (QC-passed reads + QC-failed reads)
# 950000 + 0 primary
# 980000 + 0 mapped (98.00% : N/A)
# 940000 + 0 properly paired (94.00% : N/A)

# Mapping quality distribution
samtools view aligned.bam | awk '{print $5}' | sort -n | uniq -c | sort -k2 -n

# Insert size distribution
samtools view aligned.bam | awk '$9 > 0 {print $9}' | head -1000 | sort -n | uniq -c
```

**Good alignment stats for WGS:**
- >95% mapped
- >90% properly paired
- Mean MAPQ > 50
- Median insert size = library target ± 20%

---

## Exercises

1. Align a test dataset using BWA-MEM and a small reference. Check `samtools flagstat` output.
2. Explain the CIGAR string `45M2D105M` in plain English.
3. Write a Python function that takes a SAM FLAG integer and returns a human-readable explanation.
4. You see a read with FLAG = 4. What does this mean? What should you do with it?
5. Research: What is **base quality score recalibration (BQSR)**? Why is it important before variant calling?

---

## Key Terms

| Term | Definition |
|------|------------|
| Alignment | Mapping a read to its origin position in the reference |
| CIGAR | Compact encoding of alignment operations (matches, indels, clipping) |
| SAM FLAG | Bitwise field describing read and alignment properties |
| MAPQ | Mapping quality (Phred-scaled probability of wrong placement) |
| FM-index | Compressed full-text index used by BWA / Bowtie2 |
| Seed-and-extend | Alignment strategy: find exact matches, then extend |
| Minimizer | Subsampled hash of k-mers used by Minimap2 |
| Reference genome | The consensus sequence used as an alignment target |
| Soft-clipping | Bases present in read but not used in alignment (e.g., adapter) |

---

## Next Steps

→ `09-quality-scores.md` — Per-base quality scores  
→ `12-sam-bam-cram.md` — The SAM/BAM/CRAM file format  
→ `14-samtools-deep-dive.md` — SAMtools commands  

---

*"An alignment is a hypothesis: 'this read came from here.' The CIGAR string is the evidence."*
