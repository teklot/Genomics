# 09 — Quality Scores: The Phred System

**Prerequisites:** `06-sequencing-technology.md`, `07-read-lengths-coverage.md`

**See also:** `10-quality-control.md`, `11-fasta-fastq.md`

---

## The Big Idea

Every base call from a sequencer comes with a **quality score** — a probability that the call is wrong.

```
Read:    A C T G A C T G
Qual:    I I I I I I I I
         ↑
       Q = 40 → P(error) = 0.0001 (99.99% accurate)
```

```
Good Lane:
  Q40 ████████████████████████████
  Q30 ████████████████████████████
  Q20 ████████████████████████████
  5'  ───────────────────────────→ 3'
  → All bases in green zone

Bad Lane:
  Q40 ██████████░░░░░░████████
  Q30 ███████░░░░░░██████████
  Q20 ████░░░░░░██████████████
  5'  ───────────────────────────→ 3'
  → Quality drops sharply toward 3' end

Typical Illumina 2×150 Cycle:
  • First ~5 bases: lower quality (template switching noise)
  • Middle bases: highest quality (Q35–Q40)
  • Last bases: quality degrades (phasing, dephasing effects)
```

---

## 1. The Phred Quality Score

The Phred quality score (Q) is defined as:

```
Q = -10 × log₁₀(P)

Where P = probability the base call is incorrect
```

### Score Table

| Q | P(error) | Accuracy | Interpretation |
|---|----------|----------|----------------|
| 10 | 0.1 (10%) | 90% | Very poor |
| 20 | 0.01 (1%) | 99% | Minimum for alignment |
| 30 | 0.001 (0.1%) | 99.9% | Standard minimum for variant calling |
| 40 | 0.0001 (0.01%) | 99.99% | Very good |
| 60 | 0.000001 (0.0001%) | 99.9999% | Maximum (rarely achieved) |

```python
import math

def phred_to_error(q: int) -> float:
    return 10 ** (-q / 10)

def error_to_phred(p: float) -> int:
    return int(-10 * math.log10(p))

assert phred_to_error(30) == 0.001  # 1 in 1000 chance of error
assert error_to_phred(0.001) == 30
```

### What Q30 Means

```
Q30 = 1 error per 1,000 bases

For a 30× WGS genome:
  3.2 × 10⁹ bases × 30 reads × 0.001 error rate
  = ~96 million expected base call errors

For a Q40 read (1 error per 10,000):
  3.2 × 10⁹ × 30 × 0.0001 = ~9.6 million errors
```

Alignment, consensus calling, and variant filtering will handle these errors, but the raw numbers show why quality matters.

---

## 2. Quality Score Encoding in FASTQ

Quality scores are encoded as **single ASCII characters** for compact storage.

**Phred+33 encoding** (used by Illumina 1.8+):

```
Q = ASCII_code - 33

ASCII '!'   = 33 - 33 = Q0   (P = 1.0)
ASCII '5'   = 53 - 33 = Q20  (P = 0.01)
ASCII '?'   = 63 - 33 = Q30  (P = 0.001)
ASCII 'I'   = 73 - 33 = Q40  (P = 0.0001)
ASCII 'J'   = 74 - 33 = Q41  (P = 0.00008)
```

```python
def qual_to_qscores(qual_string: str) -> list[int]:
    """Convert a FASTQ quality string to Q-score list."""
    return [ord(c) - 33 for c in qual_string]

def qscores_to_qual(qscores: list[int]) -> str:
    """Convert Q-score list back to FASTQ quality string."""
    return "".join(chr(q + 33) for q in qscores)

qual = "IIIIIII?III"
qs = qual_to_qscores(qual)
print(qs)  # [40, 40, 40, 40, 40, 40, 40, 30, 40, 40, 40]
```

### Encoding Standards

| Format | Offset | First ASCII | Range | Platforms |
|--------|--------|-------------|-------|-----------|
| Phred+33 | +33 | ! (33) | 0–41 | Illumina 1.8+, Sanger, current standard |
| Phred+64 | +64 | @ (64) | 0–40 | Illumina 1.3–1.7 (legacy) |

Most modern data uses **Phred+33**. You should explicitly verify or convert if you encounter older data.

---

## 3. Quality Score Distributions

Quality is not uniform across reads:

```
Typical Illumina 2×150:
╔══════════════════════════════════════════════════╗
║  Quality                                        ║
║  ↑                                              ║
║ 40┤   ████                                      ║
║ 30┤  ████████████                               ║
║ 20┤ ████████████████████████████████             ║
║ 10┤ ████████████████████████████████████████████ ║
║   └──────────────────────────────────────→      ║
║    1                 75               150        ║
║                        Cycle (position)          ║
╚══════════════════════════════════════════════════╝
```

**Pattern:**
- First ~5 bases: lower quality (template switching noise)
- Middle bases: highest quality (~Q35–Q40)
- Last bases: degrading quality (phasing, dephasing effects)

**Warning signs:**
- Sudden drop at a specific base → hardware issue
- Overall low quality → bad library or expired reagents
- Uniformly low quality → sample degradation

---

## 4. Base Calling Quality and Accuracy

Sequencers don't just "read" bases — they **infer** them from signals:

```
Illumina:
  Camera image → cluster intensity → position on 4-channel plot → base call + Q-score

Nanopore:
  Ionic current signal → neural network (Guppy/Dorado) → base call + Q-score

PacBio:
  Fluorescence pulses → pulse detection → base call + Q-score
```

The Q-score from the base caller tends to be **over-confident**:



This is why variant callers often recalibrate Q-scores using known truth sets.

---

## 5. Quality in Downstream Tools

### In SAM/BAM files

Each aligned read has:

- **MAPQ** (mapping quality): probability the read is placed incorrectly
- **Per-base qualities** in the QUAL field

```bash
# Check mapping quality distribution
samtools view aligned.bam | awk '{print $5}' | sort -n | uniq -c | sort -k2 -n
```

### In VCF files

Variant QUAL scores are also Phred-scaled:

```
QUAL = -10 × log₁₀(P(variant call is wrong))

QUAL=100 → P(wrong) = 10⁻¹⁰ (very confident)
QUAL=10  → P(wrong) = 0.1    (not confident)
QUAL=0   → P(wrong) = 1.0    (useless)
```

### Quality Filtering

```bash
# Filter SAM by mapping quality
samtools view -q 30 aligned.bam > high_mapq.bam

# Filter VCF by variant quality
bcftools filter -i 'QUAL > 30' variants.vcf.gz

# Filter VCF by genotype quality
bcftools filter -i 'GQ > 20' variants.vcf.gz
```

---

## 6. Quality Score Calibration

In practice, raw Q-scores are not perfectly calibrated. Tools like **GATK BaseRecalibrator** (for BQSR) adjust them:

```bash
# GATK BQSR (common in WGS pipelines)
gatk BaseRecalibrator \
    -R reference.fa \
    -I aligned.bam \
    --known-sites known_variants.vcf \
    -O recal.table

gatk ApplyBQSR \
    -R reference.fa \
    -I aligned.bam \
    --bqsr-recal-file recal.table \
    -O recalibrated.bam
```

After BQSR, the reported Q-scores more closely match actual error rates.

---

## 7. Practical Quality Commands

```bash
# Per-base quality distribution from FASTQ
zcat sample.fastq.gz | awk 'NR % 4 == 0' | \
    python3 -c "
import sys
for line in sys.stdin:
    quals = [ord(c)-33 for c in line.strip()]
    print(' '.join(map(str, quals)))
" | head -5

# Mean quality per read
zcat sample.fastq.gz | awk 'NR % 4 == 0 {
    sum=0; for(i=1;i<=length($0);i++) sum+=substr($0,i,1);
    printf \"%.2f\n\", sum/length($0)
}' | head -10

# Count reads above Q30 threshold
zcat sample.fastq.gz | awk 'NR % 4 == 0 {
    q30=0; for(i=1;i<=length($0);i++) if(substr($0,i,1)>=\"?\") q30++;
    print (q30/length($0))*100
}' | head -10
```

---

## Exercises

1. Write a function `q_to_error(q: int) -> float` and `error_to_q(p: float) -> int`.
2. A FASTQ quality line is `IIIII?IIIIIAIIII`. Convert it to Q-scores. What is the mean quality? What percentage of bases are Q30+?
3. You have a variant call with QUAL = 50. What is the probability it's wrong? Is this considered high or low confidence?
4. Research: What is the **quality score encoding** for old Solexa data? Why was Phred+64 abandoned?
5. Simulate 100 random reads at Q30. What fraction of bases would you expect to be incorrect? How many errors would that be across 100× 150 bp reads?

---

## Key Terms

| Term | Definition |
|------|------------|
| Phred score | Q = -10 × log₁₀(P_error) |
| Q30 | 99.9% accurate, 1 error per 1,000 bases |
| Phred+33 | Current FASTQ quality encoding (ASCII-33) |
| MAPQ | Mapping quality score in SAM/BAM |
| BQSR | Base quality score recalibration (GATK) |
| QUAL | Variant quality in VCF |
| GQ | Genotype quality in VCF |

---

## Next Steps

→ `10-quality-control.md` — FastQC and quality trimming  
→ `11-fasta-fastq.md` — FASTA and FASTQ format specification  
→ `14-samtools-deep-dive.md` — SAMtools commands and filtering  

---

*"Quality scores are p-values for each base. Treat them with the same skepticism."*
