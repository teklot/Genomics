# 01 — DNA: A Double-Stranded Data Structure

**Prerequisites:** None. This is the starting point.

**See also:** `02-rna.md`, `03-genes-chromosomes.md`, `11-fasta-fastq.md`

---

## The Big Idea

DNA is a **distributed, redundancy-backed, string-indexed data structure** that stores the source code of every living organism.

If you're a software engineer, think of it as a character array with strict validation rules, complementary backup copies, and a 4-character alphabet.

---

## 1. Chemical Composition

Each DNA molecule is a **polymer** — a chain of repeating units called **nucleotides**.

```
Nucleotide = [Phosphate] + [Sugar (deoxyribose)] + [Base]
```

```
        Phosphate
            |
         Sugar
            |
          Base
```

There are **four bases**, forming the four-letter alphabet:

| Base | One-Letter Code | Type |
|------|----------------|------|
| Adenine | A | Purine |
| Thymine | T | Pyrimidine |
| Cytosine | C | Pyrimidine |
| Guanine | G | Purine |

In code:

```python
from enum import Enum

class Base(Enum):
    A = "A"
    T = "T"
    C = "C"
    G = "G"
```

---

## 2. The Sugar-Phosphate Backbone

Nucleotides link together through their phosphate and sugar groups, forming a backbone:

```
Phosphate → Sugar → Phosphate → Sugar → Phosphate → Sugar ...
                |          |          |
               Base       Base       Base
```

**Analogy:** The backbone is the infrastructure layer (HTTP, TCP). The bases are the payload (JSON body).

---

## 3. Double-Stranded Structure

DNA is **double-stranded**. Two strands run in **opposite directions** (antiparallel) and are held together by **base pairing**.

### Base-Pairing Rules

```
A pairs with T (2 hydrogen bonds)
C pairs with G (3 hydrogen bonds)
```

```
Strand 1: A  T  G  C  T  A  G
             |  |  |  |  |  |
Strand 2: T  A  C  G  A  T  C
```

The two strands are **reverse complements** of each other.

```python
complement = {"A": "T", "T": "A", "C": "G", "G": "C"}

def reverse_complement(seq: str) -> str:
    return "".join(complement[b] for b in reversed(seq))

assert reverse_complement("ATGCTAG") == "CTAGCAT"
```

**Analogy:** This is like maintaining two identical servers in different data centers. If one strand is damaged, the other serves as backup.

### Directionality

DNA has a **5' (five-prime) end** and a **3' (three-prime) end**. The strands run antiparallel:

```
5' - A T G C - 3'
    | | | |
3' - T A C G - 5'
```

This direction matters for sequencing and replication.

---

## 4. The Information Is in the Sequence

The physical order of bases along the strand is the information:

```
5' - A G T G C T T A C C G A T - 3'
```

This is analogous to a binary string, but with 4 states instead of 2. Every cell in your body (except red blood cells) contains a copy of this string, roughly **3.2 billion characters** long (the human genome).

```
Human genome size: ~3.2 × 10^9 base pairs (bp)
Storage analogy: ~700 MB of uncompressed text
```

---

## 5. DNA in the Cell

DNA is **packaged** hierarchically:

```
DNA double helix (2 nm)
    → wraps around histones (nucleosomes, ~10 nm)
    → coiled into chromatin fiber (~30 nm)
    → looped and folded into chromosomes (~1 µm during division)
```

**Analogy:** Source code → files → directories → projects → repository.

---

## 6. Key Operations on DNA

Cells perform these operations constantly:

| Operation | Analogy | Enzyme |
|-----------|---------|--------|
| Replication (copy) | `git clone` | DNA polymerase |
| Transcription (read) | `cat` → pipe | RNA polymerase |
| Repair (fix) | `git revert` | DNA repair enzymes |
| Recombination (merge) | `git merge` | Recombinase |

---

## 7. The Central Dogma (Preview)

Information flows:

```
DNA  →  RNA  →  Protein
```

This is covered in detail in `04-central-dogma.md`. For now, just remember that DNA is the **permanent source of truth**.

---

## 8. Why This Matters for a Genomics Engineer

- When a sequencing machine reads DNA, it produces **strings of A/C/T/G** — you'll process these with code.
- The **reverse complement** operation shows up constantly in sequence alignment.
- Understanding directionality (5' → 3') explains why paired-end reads have a specific orientation.
- The 4-letter alphabet means compression schemes (like CRAM) can be very efficient — 2 bits per base.

---

## Exercises

1. Write a function `gc_content(seq: str) -> float` that returns the fraction of G and C bases in a DNA string.
2. Write a function `transcribe(seq: str) -> str` that replaces T with U (previewing RNA, see `02-rna.md`).
3. Generate a random 1000 bp DNA sequence. Compute its reverse complement. Verify that reverse-complementing twice returns the original.
4. Research: Why do C-G pairs have 3 hydrogen bonds while A-T pairs have 2? How does this affect DNA melting temperature?

---

## Key Terms

| Term | Definition |
|------|------------|
| Nucleotide | The monomer unit: phosphate + sugar + base |
| Base pair (bp) | Two complementary bases on opposite strands |
| Reverse complement | The strand running opposite to a given sequence |
| 5' / 3' | Directional conventions for the DNA backbone |
| Genome | The complete DNA sequence of an organism |

---

## Next Steps

→ `02-rna.md` — RNA as the transient build artifact  
→ `03-genes-chromosomes.md` — How DNA is organized into functional units  
→ `04-central-dogma.md` — The DNA → RNA → Protein pipeline  

---

*"DNA is a data structure, not a mystery."*
