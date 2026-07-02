# 04 — The Central Dogma: DNA → RNA → Protein

**Prerequisites:** `01-dna.md`, `02-rna.md`, `03-genes-chromosomes.md`

**See also:** `05-mutations-genetics.md`

---

## The Big Idea

The **central dogma** of molecular biology describes the unidirectional flow of information:

```
DNA  →  RNA  →  Protein
```

This is a **build pipeline**:

```
Source Code  →  Build Artifact  →  Executable Binary
```

---

```
┌─────────────────────────────────────────────────────────────────┐
│ NUCLEUS (Storage & Transcription)                               │
│                                                                 │
│  ┌─────────────────┐      ┌─────────────────────┐               │
│  │      DNA         │      │     pre-mRNA         │               │
│  │  Double-stranded │─────→│  Primary transcript  │               │
│  │  "Source Code"   │      │  "Build w/ comments" │               │
│  └─────────────────┘      └─────────────────────┘               │
│         │                         │                             │
│         │                   SPLICING + Nuclear Export            │
│         │                         │                             │
└─────────────────────────┬───────────────────────────────────────┘
                          │
┌─────────────────────────↓───────────────────────────────────────┐
│ CYTOPLASM (Translation)                                         │
│                                                                 │
│  ┌─────────────────────┐      ┌─────────────────┐               │
│  │      mRNA           │      │     Protein       │               │
│  │  Spliced, capped    │─────→│  Folded peptide   │               │
│  │  "Optimized Bytecode"│      │  "Executable App" │               │
│  └─────────────────────┘      └─────────────────┘               │
│         │                                                       │
│    TRANSLATION (Ribosome + tRNA)                                │
└─────────────────────────────────────────────────────────────────┘

SWE Analogy:
  Source Code  ──Compile──→  AST+Comments  ──Optimize──→  Bytecode  ──Link/Run──→  Binary/Exe
      (DNA)               (pre-mRNA)                  (mRNA)                   (Protein)

Information flows one way: DNA → RNA → Protein (Crick's original thesis)
```

## 1. The Full Pipeline

```
Step 1: TRANSCRIPTION
    DNA (nucleus)
        ↓
    pre-mRNA (nucleus)
        ↓
    mRNA processing (splicing, capping, poly-A tail)
        ↓
    mRNA (exported to cytoplasm)

Step 2: TRANSLATION
    mRNA (cytoplasm)
        ↓
    Ribosome reads codons
        ↓
    tRNA delivers amino acids
        ↓
    Polypeptide chain (amino acid sequence)
        ↓
    Protein folding
        ↓
    Functional protein
```

### Visualization

```
┌──────────────────────────────────────────────────────┐
│  NUCLEUS                                             │
│  ┌──────────┐    ┌──────────────┐    ┌──────────┐   │
│  │   DNA    │ →  │  pre-mRNA    │ →  │  mRNA    │   │
│  │ source   │    │  (w/ introns)│    │  (spliced)│   │
│  └──────────┘    └──────────────┘    └─────┬────┘   │
└────────────────────────────────────────────┼────────┘
                                             │ export
┌────────────────────────────────────────────┼────────┐
│  CYTOPLASM                                │        │
│                                           ↓        │
│  ┌──────────┐    ┌──────────────┐    ┌──────────┐  │
│  │  mRNA    │ →  │  Ribosome    │ →  │  Protein │  │
│  │          │    │  translation │    │  folded  │  │
│  └──────────┘    └──────────────┘    └──────────┘  │
└────────────────────────────────────────────────────┘
```

---

## 2. The Genetic Code: Codons → Amino Acids

The mRNA is read in **triplets** called **codons**. Each codon specifies one of 20 **amino acids**.

```text
mRNA:   A U G  |  G C U  |  U A C  |  U G A
          |         |         |         |
          ↓         ↓         ↓         ↓
AAs:    Met        Ala       Tyr      STOP
```

### The Codon Table

```
┌────────┬────────┬────────┬────────┬────────┬────────┐
│ Codon  │ AA     │ Codon  │ AA     │ Codon  │ AA     │
├────────┼────────┼────────┼────────┼────────┼────────┤
│ UUU    │ Phe    │ UCU    │ Ser    │ UAU    │ Tyr    │
│ UUC    │ Phe    │ UCC    │ Ser    │ UAC    │ Tyr    │
│ UUA    │ Leu    │ UCA    │ Ser    │ UAA    │ STOP   │
│ UUG    │ Leu    │ UCG    │ Ser    │ UAG    │ STOP   │
├────────┼────────┼────────┼────────┼────────┼────────┤
│ CUU    │ Leu    │ CCU    │ Pro    │ CAU    │ His    │
│ CUC    │ Leu    │ CCC    │ Pro    │ CAC    │ His    │
│ CUA    │ Leu    │ CCA    │ Pro    │ CAA    │ Gln    │
│ CUG    │ Leu    │ CCG    │ Pro    │ CAG    │ Gln    │
├────────┼────────┼────────┼────────┼────────┼────────┤
│ AUU    │ Ile    │ ACU    │ Thr    │ AAU    │ Asn    │
│ AUC    │ Ile    │ ACC    │ Thr    │ AAC    │ Asn    │
│ AUA    │ Ile    │ ACA    │ Thr    │ AAA    │ Lys    │
│ AUG    │ Met*   │ ACG    │ Thr    │ AAG    │ Lys    │
├────────┼────────┼────────┼────────┼────────┼────────┤
│ GUU    │ Val    │ GCU    │ Ala    │ GAU    │ Asp    │
│ GUC    │ Val    │ GCC    │ Ala    │ GAC    │ Asp    │
│ GUA    │ Val    │ GCA    │ Ala    │ GAA    │ Glu    │
│ GUG    │ Val    │ GCG    │ Ala    │ GAG    │ Glu    │
├────────┼────────┼────────┼────────┼────────┼────────┤
│ UGU    │ Cys    │ UGG    │ Trp    │ UGA    │ STOP   │
│ UGC    │ Cys    │        │        │        │        │
└────────┴────────┴────────┴────────┴────────┴────────┘

*AUG = start codon (Met = methionine)
```

**Key properties:**

- **Degenerate:** 64 codons → 20 amino acids + 3 stop codons. Multiple codons can code for the same amino acid.
- **Start:** AUG is always the start codon
- **Stop:** UAA, UAG, UGA signal termination

```python
codon_table = {
    "UUU": "F", "UUC": "F", "UUA": "L", "UUG": "L",
    "CUU": "L", "CUC": "L", "CUA": "L", "CUG": "L",
    "AUU": "I", "AUC": "I", "AUA": "I", "AUG": "M",
    "GUU": "V", "GUC": "V", "GUA": "V", "GUG": "V",
    "UCU": "S", "UCC": "S", "UCA": "S", "UCG": "S",
    "CCU": "P", "CCC": "P", "CCA": "P", "CCG": "P",
    "ACU": "T", "ACC": "T", "ACA": "T", "ACG": "T",
    "GCU": "A", "GCC": "A", "GCA": "A", "GCG": "A",
    "UAU": "Y", "UAC": "Y", "UAA": "*", "UAG": "*",
    "CAU": "H", "CAC": "H", "CAA": "Q", "CAG": "Q",
    "AAU": "N", "AAC": "N", "AAA": "K", "AAG": "K",
    "GAU": "D", "GAC": "D", "GAA": "E", "GAG": "E",
    "UGU": "C", "UGC": "C", "UGA": "*", "UGG": "W",
    "CGU": "R", "CGC": "R", "CGA": "R", "CGG": "R",
    "AGU": "S", "AGC": "S", "AGA": "R", "AGG": "R",
    "GGU": "G", "GGC": "G", "GGA": "G", "GGG": "G",
}

def translate(rna: str) -> str:
    protein = []
    for i in range(0, len(rna) - 2, 3):
        codon = rna[i:i+3]
        aa = codon_table.get(codon, "?")
        if aa == "*":
            break
        protein.append(aa)
    return "".join(protein)

assert translate("AUGGCUUACUGA") == "MAY"
```

---

## 3. Reading Frames

A given RNA can be read in **3 forward reading frames**:

```
RNA:    A U G G C U U A C U G A
Frame1: AUG | GCU | UAC | UGA     →  Met-Ala-Tyr-STOP
Frame2:  AUG | GCU | UAC | UGA    →  none (no start)
Frame3:   AUG | GCU | UAC | UGA   →  none (no start)
```

And 3 reverse frames from the complement.

Finding the correct reading frame is like finding the right entry point in a binary — the ribosome scans for the first AUG start codon.

---

```
                    ┌───────────────────────────┐
                    │      Ribosome (Large)      │
                    │  ┌─────┐ ┌─────┐ ┌─────┐  │
                    │  │  E  │ │  P  │ │  A  │  │
                    │  │Exit │ │Pepti│ │Amino│  │
                    │  └─────┘ └─────┘ └─────┘  │
   5' ── AUG ─── GCU ─── UAC ─── UGA ────────── 3'
                    │      Ribosome (Small)      │
                    └───────────────────────────┘
                          ────→ movement 5' → 3'

  Growing peptide: Met─Ala─Tyr─...
                       ↑
            ┌──────────────┐
            │  tRNA (UAC)  │── Incoming: Tyr
            └──────────────┘

  The 3-Stage Pipeline (CPU-in-Cell Analogy):
    A site: new tRNA binds (fetch)
    P site: peptide held (execute)
    E site: tRNA exits (write-back)
```

## 4. Translation Machinery

```
                    Ribosome
               ┌───────────────────────────┐
               │   A site   P site  E site  │
               │  ┌─────┐ ┌─────┐ ┌─────┐ │
    mRNA  ──→  │  │ UAC │ │ AUG │ │ GCU │ │ ──→  growing
               │  └─────┘ └─────┘ └─────┘ │      peptide
               └───────────────────────────┘
                       ▲         ▲
                       │         │
                     tRNA       tRNA
                   (anticodon  (with Met)
                    = AUG)
```

| Ribosome Site | Function |
|---------------|----------|
| A (aminoacyl) | Incoming tRNA with amino acid |
| P (peptidyl) | Holds the growing polypeptide |
| E (exit) | Empty tRNA exits |

**Analogy:** The ribosome is a 3-stage CPU pipeline: fetch → decode → execute.

---

## 5. Protein Folding — The Final Step

After translation, the linear amino acid chain folds into a 3D structure:

```
Primary structure:    Met-Ala-Tyr-Pro-Gly-...   (linear sequence)
Secondary structure:  α-helices, β-sheets        (local folding)
Tertiary structure:   3D shape of one chain      (global folding)
Quaternary structure: Multiple chains together   (complex)
```

**Analogy:** This is like a binary going through linking, optimization, and deployment.

**Why it matters:**
- A protein's function depends on its 3D shape
- A single amino acid change can destroy the fold → disease (sickle cell, cystic fibrosis)
- This is why **missense mutations** (see `05-mutations-genetics.md`) can be devastating

---

## 6. The Complete Software Analogy

```
┌───────────────────────────────┬────────────────────────────┐
│ Biology                       │ Software Engineering        │
├───────────────────────────────┼────────────────────────────┤
│ DNA                           │ Source code repository     │
│ Gene                          │ A function / module        │
│ Promoter                      │ Function signature / entry │
│ Transcription                 │ Compiler frontend          │
│ pre-mRNA (with introns)       │ AST with comments          │
│ Splicing                      │ Dead code elimination      │
│ mRNA                          │ Optimized IR / bytecode    │
│ Ribosome                      │ CPU / interpreter          │
│ Codon                         │ Machine instruction        │
│ tRNA                          │ Operand / constant table   │
│ Amino acid                    │ Primitive operation        │
│ Polypeptide chain             │ Assembly output            │
│ Protein folding               │ Binary linking             │
│ Functional protein            │ Running executable         │
│ Post-translational mods       │ Hot patching / plugins     │
│ Degradation (proteasome)      │ Garbage collection         │
└───────────────────────────────┴────────────────────────────┘
```

---

## 7. Practical Implications

- **Synonymous mutations** (codon changes that don't change the amino acid) are often neutral
- **Nonsense mutations** (premature stop codons) usually break the protein
- **Alternative splicing** means one gene can produce many proteins
- **Reading frame** is critical: a single base insertion shifts everything downstream

These concepts directly impact how you interpret VCF files and variant annotations.

---

## Exercises

1. Write a function that translates an arbitrary DNA sequence to protein (first transcribe DNA→RNA, then translate).
2. Write a function that finds all open reading frames (ORFs) in a DNA sequence — regions between a start codon (ATG) and a stop codon.
3. Research: What is the longest protein-coding gene in the human genome? How many amino acids does it encode?
4. Given the DNA sequence `ATGGCCGGCGCTTTGA`, transcribe it to RNA, then translate to protein using the codon table. What is the final amino acid sequence?

---

## Key Terms

| Term | Definition |
|------|------------|
| Codon | A triplet of RNA bases specifying one amino acid |
| Start codon | AUG (methionine) — begins translation |
| Stop codon | UAA, UAG, UGA — terminates translation |
| Reading frame | The grouping of codons (3 possible frames) |
| Open reading frame (ORF) | A sequence with start → codons → stop |
| Polypeptide | A chain of amino acids (unfolded protein) |
| Translation | mRNA → protein synthesis |
| Central dogma | DNA → RNA → Protein information flow |

---

## Next Steps

→ `05-mutations-genetics.md` — When the source code changes  
→ `06-sequencing-technology.md` — How we read the source code  

---

*"The central dogma is a build pipeline. Debug it like one."*
