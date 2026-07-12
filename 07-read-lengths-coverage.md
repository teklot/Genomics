# 07 — Read Lengths & Sequencing Coverage

**Prerequisites:** `06-sequencing-technology.md`

**See also:** `08-alignment-theory.md`, `09-quality-scores.md`

---

## The Big Idea

**Read length** determines what kinds of variants you can detect. **Coverage** determines how confident you can be in your base calls.

Together, they define the fundamental trade-offs in every sequencing experiment:

```
Low coverage + long reads = structural variants, poor SNV sensitivity
High coverage + short reads = accurate SNVs, poor structural variant detection
```

---

## 1. Read Length

A **read** is one contiguous DNA sequence produced by the sequencer.

```
Read length (bp) = number of bases reported
```

### Examples

| Platform | Typical Read Length | What It Can Detect |
|----------|-------------------|-------------------|
| Illumina short read | 2×150 bp | SNVs, small indels (<50 bp) |
| Illumina long mate-pair | 2×8 kb | Large structural variants |
| PacBio HiFi (CCS) | 15–25 kb | All variant types |
| Nanopore (R10.4) | 30–100+ kb | All variant types, phasing |
| Sanger | 600–1,000 bp | Targeted variant confirmation |

### Why Read Length Matters

**Short reads** work well for:

- SNV detection
- Small indels
- Gene expression quantification (RNA-seq)
- Known variant screening

**Short reads struggle with:**

- Repetitive regions (e.g., centromeres, telomeres)
- Structural variants (can't span the breakpoints)
- Phasing (which parent contributed which variant)
- De novo genome assembly (lots of gaps)

**Long reads** excel at:

- Structural variant detection
- Resolving repeats
- Phasing (haplotype-resolved genomes)
- De novo assembly
- Transcript isoform detection (Iso-Seq)

### The mappability problem

A read that maps to multiple locations is **multimapped** and usually discarded.

```
Read length 50 bp: unique in ~70% of genome
Read length 100 bp: unique in ~85% of genome
Read length 150 bp: unique in ~92% of genome
```

Short reads in repetitive regions are like searching for a 3-word phrase in a book — too short to be unique.

---

## 2. Coverage (Depth)

**Coverage** measures how many times each base in the genome is sequenced.

```
Coverage = (Total bases sequenced) / (Genome size)

Example:
  90 Gb sequenced ÷ 3 Gb human genome = 30× coverage
```

### Coverage Naming

| Term | Depth | Example Use |
|------|-------|-------------|
 | Low-pass WGS | 1–5× | Population genetics, ancestry |
| Standard WGS | 30× | Clinical germline variant detection |
| High-coverage WGS | 60×+ | Somatic mutation detection (cancer) |
| Ultra-deep | 500×+ | ctDNA liquid biopsy |
| Whole exome | 100–200× | Coding region focused analysis |

### Coverage Distribution

```
Reference: ──────────────────────────────────────────────

Reads stacked above (high to medium to low coverage):
 ████████████
 ████████████
 ███████████
 ███████████  ███████
 ██████████   ██████
 ██████████   ██████   █████
 █████████    ██████   ████
 █████████    █████    ███
 ████████     █████    ███
 ──────────────────────────────
   9× depth   4× depth 2× depth

Mean coverage = 5× (each base read ~5 times on average)
Higher coverage = more confidence in base calls & variant detection
```

Coverage is not uniform across the genome:

```
Position: ████████████████░░░░██████████████████░░██████
Coverage:    45×         2×         50×          8×
```

Regions with:

- **High GC content** → often low coverage (sequencing bias)
- **Low GC content** → often low coverage
- **Repetitive regions** → may have inflated coverage (multimapped reads)
- **Centromeres** → near-zero coverage (too repetitive)

```
P(k) = (λ^k × e^(-λ)) / k!    where λ = coverage depth

Coverage distribution widens at lower depths:
  λ=5×  (low-pass)  → wide spread, ~7% genome with 0× coverage
  λ=15× (exome)     → narrower,  <1% zero-coverage
  λ=30× (WGS)       → tight, <0.01% below 10×, 99.9% SNV confidence

Example probabilities:
  k     λ=5×    λ=15×   λ=30×
  0     0.007   3e-7    9e-14
  5     0.175   0.002   0.000
 10     0.018   0.048   0.000
 15     0.000   0.102   0.001
 20     0.000   0.042   0.013
 30     0.000   0.000   0.073
```

### The Lander-Waterman Model

The distribution of reads across the genome follows a **Poisson distribution**:

```
P(k) = (λ^k × e^(-λ)) / k!

Where:
  λ = coverage depth
  P(k) = probability a base is covered by exactly k reads
```

At 30× coverage:

```
P(0) = 0.000000  →  ~0% of genome with zero coverage
P(1) = 0.000001  →  negligible
P(10) = 0.002    →  0.2%
P(25) = 0.04     →  4%
P(30) = 0.04     →  4%
P(35) = 0.04     →  4%
```

```python
import math

def poisson_prob(k: int, lam: float) -> float:
    return (lam**k * math.exp(-lam)) / math.factorial(k)

# Probability a base at 30× has coverage < 10
prob_low = sum(poisson_prob(k, 30) for k in range(10))
print(f"P(coverage < 10 at 30×) = {prob_low:.6f}")  # ~0.00003
```

### Why 30×?

30× WGS is the standard because:

- ~99.9% of the genome is covered by at least 10 reads
- 10+ reads are needed for reliable heterozygous SNV detection
- Above 30×, you get diminishing returns for SNVs (but may want more for SVs)

---

## 3. Coverage and Variant Detection

### Minimum Coverage for Variant Calling

```
Germline heterozygous (VAF ~0.5):  need ≥10× total, ≥3 variant reads
Germline homozygous (VAF ~1.0):    need ≥8× total
Somatic (VAF ~0.05–0.3):           need ≥50–100× depth
```

### Depth vs Allele Frequency Detection

```
At 30× coverage:
  Can detect variants with VAF ≥ ~0.20 reliably
  For VAF 0.05 (typical liquid biopsy): need ≥500×
```

### Coverage Impact on Genotype Quality

```python
# At low coverage, heterozygous calls are unreliable
# Likelihood of observing 0 variant reads from a true heterozygous site at depth d:
import math

def prob_miss_het(depth: int, vaf: float = 0.5) -> float:
    return (1 - vaf) ** depth

for d in [5, 10, 20, 30]:
    pm = prob_miss_het(d)
    print(f"Depth {d}×: P(missing het) = {pm:.4f}")

# Output:
# Depth 5×:  P(missing het) = 0.0313
# Depth 10×: P(missing het) = 0.0010
# Depth 20×: P(missing het) = 0.0000
# Depth 30×: P(missing het) = 0.0000
```

---

## 4. Coverage and Sequencing Cost

```
Cost for one human genome:

  Coverage          FASTQ       BAM         Cost (approx.)
  5× (low-pass):    30 GB       10 GB       $100–200
  30× (WGS):        200 GB      60 GB       $600–1,000
  60× (deep):       400 GB      120 GB      $1,200–2,000
```

### Coverage Downsampling

You can intentionally downsample a dataset:

```bash
# Downsamples a BAM to 20× mean coverage
samtools view -s 0.67 -b high_coverage.bam > downsample_20x.bam
```

---

## 5. Practical Coverage Commands

```bash
# Mean coverage from a BAM
samtools depth sample.bam | awk '{sum+=$3; count++} END {print sum/count}'

# Coverage per chromosome
samtools depth -r chr1 sample.bam | awk '{sum+=$3; count++} END {print "chr1:", sum/count}'

# Fraction of genome at ≥10× (using mosdepth)
mosdepth -t 4 sample sample.bam
# then check sample.per-base.bed.gz

# Depth at a specific position
samtools depth -r chr1:1000000-1000100 sample.bam

# Coverage histogram
samtools depth sample.bam | cut -f3 | sort -n | uniq -c | head -50
```

---

## 6. Insert Size (Paired-End)

For paired-end reads, **insert size** is the distance between the two reads on the original fragment:

```
          ← insert size →
          ┌─────────────────┐
Read 1:   └────→            │
Read 2:    ◄────────────────┘
```

```bash
# Distribution of insert sizes
samtools view sample.bam | awk '{print sqrt($9^2)}' | sort -n | uniq -c
```

Insert size outliers can indicate structural variants:

- **Larger than expected** → deletion in the sample
- **Smaller than expected** → insertion in the sample
- **Discordant orientation** → inversion or translocation

---

## Exercises

1. A NovaSeq S4 flow cell produces 10B reads at 2×150 bp. How many human genomes at 30× can be sequenced on one flow cell?
2. At 30× coverage, what percentage of the genome has ≥20× coverage? Use the Poisson formula.
3. Write a script that simulates a coverage file (random Poisson-distributed per-base depths with λ=30) and reports the percentage of bases at ≥10×, ≥20×, ≥30×.
4. At 30× coverage with a Poisson distribution, what is the probability that a given base has zero coverage? How many bases in the human genome (~3.2 Gb) would you expect to have no coverage?
5. Research: What is **GC bias** and how does it affect coverage? How do library preparation methods try to mitigate it?

---

## Key Terms

| Term | Definition |
|------|------------|
| Read length | Number of bases in a single sequencing read |
| Coverage (depth) | Average number of reads covering each base |
| Lander-Waterman | Poisson model for read distribution |
| Insert size | Distance between paired-end reads on the fragment |
| GC bias | Underrepresentation of extreme-GC regions |
| Downsampling | Reducing coverage by discarding reads |
| Multimapped | Read that aligns equally well to multiple positions |

---

## Next Steps

→ `08-alignment-theory.md` — How reads are placed on the reference genome  
→ `09-quality-scores.md` — Per-base quality scores  

---

*"Coverage is like sample size in a survey — more is almost always better, but the cost curve is exponential."*
