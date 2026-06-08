# 05 — Mutations & Genetics: When the Source Code Changes

**Prerequisites:** `01-dna.md`, `02-rna.md`, `03-genes-chromosomes.md`, `04-central-dogma.md`

**See also:** `13-vcf-bcf.md`, `15-bcftools-deep-dive.md`

---

## The Big Idea

A **mutation** is a change in the DNA sequence. In software terms, it's a **git diff** against the reference genome.

```
Reference: ATGCGTACCCGA
Sample:    ATGAGTACCCGA
                  ^
                C→A substitution
```

---

## 1. Types of Mutations by Scale

### Single Nucleotide Variants (SNVs)

A single base change:

```
Reference:  A T G C G T
Mutant:     A T G A G T
                  ^
```

| Type | Effect |
|------|--------|
| **Transition** | Purine ↔ Purine or Pyrimidine ↔ Pyrimidine (A↔G, C↔T) |
| **Transversion** | Purine ↔ Pyrimidine (A↔C, A↔T, G↔C, G↔T) |

Transitions are ~2× more common than transversions. The **Ti/Tv ratio** (~2.1 for whole genome) is a standard QC metric for variant calls.

### Insertions and Deletions (Indels)

```
Reference:  A T G C G T A A
Insertion:  A T G C G T A A A A    (+2 bp)
Deletion:   A T G C — T A A        (-1 bp)
```

Indels are harder to detect than SNVs because they shift the alignment.

### Structural Variants (SVs)

Larger changes affecting 50+ bp:

```
Deletion:     ┌──────────────────┐
              │  50,000 bp lost  │
              └──────────────────┘

Duplication:  ┌──────────────────┐
              │  block repeated  │
              └──────────────────┘
              ┌──────────────────┐
              │  block repeated  │
              └──────────────────┘

Inversion:    ┌──────────────────┐
              │  flipped around  │
              └──────────────────┘

Translocation:
  chr1 ────┐
           │  swap between chromosomes
  chr2 ────┘
```

---

## 2. Types of Mutations by Effect on Protein

```
DNA:    A T G  G C T  T A C  T G A
        Met    Ala    Tyr    STOP (reference)

                          ↓

     Missense (single AA change):
        A T G  G C T  T C C  T G A
        Met    Ala    Ser    STOP        ← Tyr→Ser

     Nonsense (premature stop):
        A T G  G C T  T A A  T G A
        Met    Ala    STOP                ← truncated!

     Silent (codon still codes same AA):
        A T G  G C G  T A C  T G A
        Met    Ala    Tyr    STOP        ← no change (GCU→GCG)

     Frameshift (indel shifts everything):
        A T G  G C T  T A C  T G A
        A T G  G C T  A C T  G A
        Met    Ala    Thr               ← every codon after is wrong!
```

| Variant Type | Effect on Protein | Severity |
|-------------|-------------------|----------|
| Synonymous (silent) | No change | Usually none (but can affect splicing) |
| Missense | Single AA change | Variable (benign to pathogenic) |
| Nonsense | Premature stop codon | Usually severe (truncated protein) |
| Frameshift | All downstream AAs changed | Almost always severe |
| In-frame indel | AA(s) added/removed, frame intact | Variable |

---

## 3. Germline vs Somatic Mutations

```
Germline: Present in every cell, inherited from parent
          → Found in all tissues
          → Passed to offspring
          → Detectable in blood DNA

Somatic:  Arises in one cell during life
          → Found only in affected tissue
          → Not inherited / not passed on
          → Requires sequencing of the affected tissue
```

**Analogy:**

- **Germline mutation** = bug introduced into the initial commit, present in every fork
- **Somatic mutation** = runtime bug that only appears under specific conditions on one node

### Clinical Impact

| | Germline | Somatic |
|---|---|---|
| **Cancer** | Inherited susceptibility (e.g., BRCA1) | Driver mutations in tumors |
| **Diagnosis** | Whole-genome or whole-exome sequencing of blood | Tumor biopsy sequencing |
| **VAF** (variant allele frequency) | ~50% (heterozygous) or ~100% (homozygous) | Variable (0.1%–90%) |

**Variant allele frequency (VAF)** = fraction of reads supporting the variant:

```
VAF = variant_reads / total_reads

Germline heterozygous: ~0.5 (50% of reads show the variant)
Germline homozygous:   ~1.0 (100% of reads)
Somatic (pure tumor):  ~0.1–0.8 (depends on tumor purity)
```

---

## 4. Hardy-Weinberg Principle

For population genetics, the **Hardy-Weinberg equilibrium** describes expected genotype frequencies:

```
Given allele A frequency = p
Given allele a frequency = q
where p + q = 1

Expected:
  AA = p²
  Aa = 2pq
  aa = q²
```

**Example:** If a variant has allele frequency 0.1 (10% of chromosomes carry it):

```
  AA (homozygous reference): 0.9² = 0.81 (81%)
  Aa (heterozygous):         2 × 0.9 × 0.1 = 0.18 (18%)
  aa (homozygous variant):   0.1² = 0.01 (1%)
```

Deviations from HWE can indicate **genotyping errors**, **population structure**, or **selection**.

---

## 5. dbSNP and Variant Databases

| Database | Content | Use Case |
|----------|---------|----------|
| **dbSNP** | Known SNVs and small indels | Filtering common variants |
| **ClinVar** | Variants with clinical significance | Interpretation |
| **gnomAD** | Population allele frequencies | Filtering by rarity |
| **COSMIC** | Somatic mutations in cancer | Cancer research |
| **1000 Genomes** | Global population variation | Frequency checking |

**Typical workflow:**

1. Call variants → VCF
2. Annotate with **dbSNP ID**, **gnomAD frequency**, **ClinVar significance**
3. Filter: keep rare (AF < 1%) non-synonymous variants
4. Prioritize: focus on known pathogenic or novel missense/nonsense variants

---

## 6. Zygosity and Genotype Encoding

In VCF, genotypes are encoded as:

```
GT:PL:GQ
0/1:50,0,255:50
```

| Code | Meaning | VAF expected |
|------|---------|-------------|
| 0/0 | Homozygous reference | ~0 |
| 0/1 | Heterozygous | ~0.5 |
| 1/1 | Homozygous variant | ~1.0 |
| 0/2 | Heterozygous with second alt allele | variable |
| ./. | No call (low confidence) | — |

The **PL** field stores **Phred-scaled likelihoods** (see `09-quality-scores.md`):

```
PL: 50,0,255
    ↑   ↑   ↑
  0/0 0/1 1/1  (lower = more likely)
```

---

## 7. Practical Mutation Analysis Commands

```bash
# Count variant types in a VCF
bcftools stats sample.vcf.gz | grep "SN"

# Filter to rare non-synonymous variants
bcftools filter -i 'INFO/AF < 0.01 && INFO/ANN ~ "missense"' sample.vcf.gz

# Calculate Ti/Tv ratio
bcftools stats sample.vcf.gz | grep "TSTV"

# Find de novo mutations (child variant not in parents)
bcftools isec -C child.vcf.gz mother.vcf.gz father.vcf.gz
```

---

## Exercises

1. Write a function that classifies a mutation as transition or transversion.
2. Write a function that takes a reference sequence and a variant (position, ref, alt) and returns the mutated protein sequence.
3. Given a VAF of 0.32, is this more likely germline heterozygous or somatic? Why?
4. Research: What is the **CpG island** effect? Why are C→T transitions more common in CpG contexts?

---

## Key Terms

| Term | Definition |
|------|------------|
| SNV / SNP | Single nucleotide variant / polymorphism |
| Indel | Insertion or deletion of bases |
| SV | Structural variant (>50 bp) |
| Missense | Mutation changing one amino acid |
| Nonsense | Mutation creating a premature stop codon |
| Frameshift | Indel that shifts the reading frame |
| Germline | Inherited, present in all cells |
| Somatic | Acquired, present only in some cells |
| VAF | Variant allele frequency |
| Ti/Tv ratio | Transition / transversion ratio (~2.1 for WGS) |
| Zygosity | Homozygous (0/0 or 1/1) vs heterozygous (0/1) |
| HWE | Hardy-Weinberg equilibrium |

---

## Next Steps

→ `06-sequencing-technology.md` — How sequencing machines read DNA  
→ `08-alignment-theory.md` — How reads are mapped to the reference  
→ `13-vcf-bcf.md` — The VCF file format for storing variants  

---

*"A mutation is just a git diff — but the consequences can be more serious than a merge conflict."*
