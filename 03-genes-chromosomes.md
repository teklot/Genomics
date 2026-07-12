# 03 — Genes & Chromosomes: Organizing the Source Code

**Prerequisites:** `01-dna.md`, `02-rna.md`

**See also:** `04-central-dogma.md`, `05-mutations-genetics.md`

---

## The Big Idea

If the full genome is a **monorepo** of 3.2 billion characters, then **chromosomes** are the top-level directories and **genes** are the individual modules or services.

---

## 1. Chromosomes — The Top-Level Directories

Humans have **23 pairs** of chromosomes (46 total):

- 22 pairs of **autosomes** (numbered 1–22 by size, largest first)
- 1 pair of **sex chromosomes** (XX = female, XY = male)

```
Chromosome 1:  ~248 Mbp  (largest)
Chromosome 22: ~50 Mbp   (smallest autosome)
Chromosome Y:  ~57 Mbp   (smallest overall)
Chromosome X:  ~156 Mbp
```

**Analogy:** A monorepo with 23 top-level directories. Each directory is labeled `chr1/`, `chr2/`, ..., `chr22/`, `chrX/`, `chrY/`.

```python
chromosomes = {
    "chr1": 248_956_422,   # length in base pairs
    "chr2": 242_193_529,
    # ... etc
    "chr22": 50_818_468,
    "chrX": 156_040_895,
    "chrY": 57_227_415,
}
```

### Ploidy

- **Diploid:** Two copies of each chromosome (one from each parent)
- **Haploid:** One copy (sperm and egg cells)
- **Polyploid:** More than two copies (common in plants, rare in humans)

For genomic analysis, this means:

- A position on chromosome 1 typically has **two alleles** (one maternal, one paternal)
- A variant caller must distinguish between **homozygous** (both copies same) and **heterozygous** (copies differ)

---

```
           Tel                Tel
            │                 │
    p arm ──┤  ┌────────────┐ ├── p arm
  (Short)   │  │ Centromere │ │   (Short)
            │  └────────────┘ │
    q arm ──┤                 ├── q arm
  (Long)    │                 │   (Long)
            │                 │
           Tel                Tel
          Sister Chromatids

  • p = "petit" (short arm)
  • q = follows p in alphabet (long arm)
  • Centromere: region where chromatids join
  • Telomeres: protective caps at chromosome ends
  • Humans have 23 pairs of chromosomes
```

## 2. Chromosome Structure

A chromosome is not just a long DNA string. It has:

```
Telomere ─ Centromere ─ Telomere
    |          |           |
    [p arm]  [q arm]   [p arm]
```

| Region | Function | Genomic Analogy |
|--------|----------|-----------------|
| **Telomere** | Protective caps at ends | `\n` file terminator |
| **Centromere** | Attachment point for cell division | Partition key |
| **p arm** (short) | Gene-rich region | Hot module |
| **q arm** (long) | Gene-rich region (often larger) | Another hot module |

**Gene density varies massively** across chromosomes:

- Chromosome 19: ~23 genes per Mbp (dense)
- Chromosome Y: ~1 gene per Mbp (sparse)
- Chromosome 13: ~5 genes per Mbp (sparse)

---

## 3. Karyotype

A **karyotype** is a visual representation of all chromosomes, arranged by size:

```
     1      2      3      4      5
   ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐
   │   │  │   │  │   │  │   │  │   │
   └───┘  └───┘  └───┘  └───┘  └───┘

     6      7      8      9     10
   ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐
   │   │  │   │  │   │  │   │  │   │
   └───┘  └───┘  └───┘  └───┘  └───┘

    11     12     13     14     15
   ...

   (Full human karyotype: 22 pairs + X/Y)
```

---

## 4. Genes — The Functional Units

A **gene** is a segment of DNA that encodes a functional product (usually a protein).

**Analogy:**

```
Repository = Genome     (~3.2 GB)
Directory  = Chromosome (~50–250 MB)
File       = Gene       (~1–100 KB)
```

### How Many Genes?

| Category | Count |
|----------|-------|
| Protein-coding genes | ~20,000 |
| Non-coding RNA genes | ~20,000 |
| Pseudogenes (broken copies) | ~14,000 |
| **Total genes** | ~60,000 |

That's only ~1.5% of the genome that codes for proteins.

```
Direction of transcription (5' → 3')
──>───────────────────────────────────────────────────────────────────────────>

┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ Promoter │ │  5' UTR  │ │  Exon 1  │ │ Intron 1 │ │  Exon 2  │ │ Intron 2 │ │  Exon 3  │ │  3' UTR  │
└──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘
                                                                                         
═══════════════════════════════════════════════════════════════════════════════════════════
DNA Double Strand

After Splicing (Mature mRNA):
  [5' UTR] ── [Exon 1] ── [Exon 2] ── [Exon 3] ── [3' UTR]
  Introns removed. 5' cap and poly-A tail added.

Legend:
  ███ Exon (coding)    ░░░ Intron (removed)    ▓▓▓ Regulatory
  SWE Analogy: Gene = Source File, Exons = Included Modules, Introns = Comments/Debug Code
```

### Gene Structure

```
Promoter   Exon 1   Intron 1   Exon 2   Intron 2   Exon 3   Terminator
    ↓        ↓                   ↓                   ↓
   ┌────┬─────────┬──────────┬─────────┬──────────┬─────────┬────┐
   │    │  E1     │  I1      │  E2     │  I2      │  E3     │    │
   └────┴─────────┴──────────┴─────────┴──────────┴─────────┴────┘
```

| Region | Purpose |
|--------|---------|
| **Promoter** | "Start button" — where RNA polymerase binds |
| **Exon** | Coding sequence (kept in final mRNA) |
| **Intron** | Intervening sequence (spliced out) |
| **5' UTR** | Untranslated region before start codon |
| **3' UTR** | Untranslated region after stop codon |
| **Terminator** | "Stop button" — signals end of transcription |

### Gene Naming Conventions

| Style | Example |
|-------|---------|
| Official symbol | **TP53** (tumor protein p53) |
| Full name | "tumor protein p53" |
| Aliases | P53, TRP53 |
| Ensembl ID | ENSG00000141510 |

You'll encounter all of these in VCF files and annotation databases.

---

## 5. The Remaining 98.5% of the Genome

Only ~1.5% of the human genome codes for proteins. What's the rest?

| Component | Fraction | Function |
|-----------|----------|----------|
| Introns | ~25% | Removed during splicing |
| Intergenic regions | ~60% | Regulatory, structural |
| Repetitive elements | ~45% | Transposons, repeats "junk" DNA |
| Regulatory regions | ~8% | Promoters, enhancers, silencers |

**Non-coding DNA is not "junk"** — much of it has regulatory function:

- **Enhancers** increase transcription (think: config flags)
- **Promoters** initiate transcription (think: entry points)
- **Silencers** decrease transcription (think: rate limiters)
- **Insulators** block enhancer effects (think: access control lists)

---

## 6. Key Coordinate Systems

When referring to a position in the genome:

```
GRCh38/hg38 — chr1:10,000,000-10,010,000

chr1     = chromosome
10,000,000 = start position (1-indexed)
10,010,000 = end position (1-indexed, inclusive)
```

**Genome builds** change over time:

| Build | Release | Used by |
|-------|---------|---------|
| GRCh37/hg19 | 2009 | Legacy pipelines (still common) |
| GRCh38/hg38 | 2013 | Current standard |
| T2T-CHM13 | 2022 | Complete (no gaps) |

A variant called at "chr1:10000000" in hg19 may be at "chr1:9998500" in hg38. This is called **liftover**.

```bash
# LiftOver with CrossMap
crossmap.py liftover hg19ToHg38.over.chain.gz input.bed output.bed
```

---

## 7. Genome Assembly

The reference genome is assembled from many individuals. It's not a single person's genome — it's a **consensus**.

```
Sanger sequencing     →  ~100Kb contigs  (2001, draft)
BAC sequencing        →  ~1Mb contigs    (2003, finished)
Next-gen sequencing   →  Contigs + gaps  (GRCh38, 2013)
Telomere-to-telomere  →  No gaps         (T2T-CHM13, 2022)
```

---

## Exercises

1. Look up the length of human chromosome 1 from the GRCh38 genome build. What is it in base pairs? How does it compare to chromosome 21?
2. Find the genomic coordinates of the **TP53** gene on GRCh38. What are its start/end positions on chr17? How many exons does it have?
3. Research: What is the difference between a **gene** and a **pseudogene**? Given a DNA sequence, how could you detect if it's a pseudogene?
4. Write a function that, given a chromosome length and a position, determines whether it's on the p arm or q arm (you'll need to look up centromere positions).

---

## Key Terms

| Term | Definition |
|------|------------|
| Chromosome | A single, long DNA molecule containing many genes |
| Gene | A DNA segment encoding a functional product |
| Exon | Coding region retained in the spliced mRNA |
| Intron | Non-coding region removed during splicing |
| Promoter | DNA region upstream of a gene that initiates transcription |
| Karyotype | Visual arrangement of chromosomes |
| Diploid | Having two copies of each chromosome |
| Genome build | A versioned assembly of the reference genome |
| Locus | A specific genomic position |

---

## Next Steps

→ `04-central-dogma.md` — How genes are expressed: transcription → translation → protein  
→ `05-mutations-genetics.md` — When the source code changes  

---

*"The genome is a monorepo where most files are configuration, not code."*
