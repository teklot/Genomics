# 11 — FASTA & FASTQ: Input/Output Formats

**Prerequisites:** `06-sequencing-technology.md`, `09-quality-scores.md`, `10-quality-control.md`

**See also:** `12-sam-bam-cram.md`, `14-samtools-deep-dive.md`

---

## The Big Idea

**FASTA** stores reference sequences (genomes, proteins). **FASTQ** stores sequencing reads with quality scores.

```
FASTA:   The reference library (read-only, authoritative)
FASTQ:   The experimental data (noisy, redundant)
```

---

## 1. FASTA Format

### Structure

```
>sequence_id description (optional)
ATCGGCTAACGTAGCCTAGCTAGCATCG
ATCGCTACGTAGCGATCGATCGTAGCT
ACGTAGCATCGATCGATCGATCGTAGC
```

- **Header line** starts with `>`
- **Sequence lines** follow (any line length, typically 60–80 characters)
- Multiple sequences in one file (separated by `>` headers)

### Examples

```fasta
>chr1 GRCh38.p14 chromosome 1, GRCh38 Primary Assembly
GGCTCACTTCCACCGCTTCTCTTTCTGCACAGCTAGTTCAGAATTCG
AGAGCCCTCAGGTTCCTCCTCGTTTTCTAAGGGTTAACTGCTGTG
...

>chrM human mitochondrion, Revised Cambridge Reference Sequence
GATCACAGGTCTATCACCCTATTAACCACTCACGGGAGCTCTCCATGCAT
...
```

### Protein FASTA

```fasta
>sp|P04637|P53_HUMAN Tumor suppressor p53
MEEPQSDPSVEPPLSQETFSDLWKLLPENNVLSPLPSQAMDDLMLSPDDIEQWFTEDPGP
DEAPRMPEAAPPVAPAPAAPTPAAPAPAPSWPLSSSVPSQKTYQGSYGFRLGFLHSGTAK
```

### FASTA Index (.fai)

For large references, samtools creates an index for random access:

```bash
# Create .fai index
samtools faidx genome.fa

# Extract a specific region
samtools faidx genome.fa chr1:1000000-1000100

# The .fai file looks like:
# chr1    248956422    6    50    51
# chr2    242193529    249264421    50    51
#
# Columns: name, length, offset, line_bases, line_width
```

### When You'll Use FASTA

```
- Reference genome files (hg38.fa, mm10.fa)
- Protein sequence databases
- Transcriptome sequences
- Custom target capture regions
- Primers and probes
```

---

## 2. FASTQ Format

### Structure

```
@NB500:1:1101:1117:1967#0/1         ← Header & Metadata
ACTGACTGACTGACTGACTGACTGACTGACTG    ← Raw Sequence (Bases)
+                                   ← Separator
IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII    ← Quality (Phred+33)
```

A FASTQ record has **4 lines per read**:

```
@READ_ID  (header — starts with @)
SEQUENCE  (A/C/T/G/N)
+         (separator — usually a repeat of the header)
QUALITY   (quality scores encoded as ASCII)
```

### Example

```
@NB500:1:1101:1117:1967#0/1
ACTGACTGACTGACTGACTGACTGACTGACTGACTGACTGACTGACTGACTGACTGACTGA
+
IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII
```

### FASTQ Header Fields (Illumina)

```
@NB500:1:1101:1117:1967#0/1
  │     │ │    │    │   │ │
  │     │ │    │    │   │ └─read 1 or read 2
  │     │ │    │    │   └───index (multiplexing)
  │     │ │    │    └───────x-coordinate on flow cell
  │     │ │    └───────────y-coordinate on flow cell
  │     │ └────────────────tile number
  │     └──────────────────flow cell lane
  └────────────────────────instrument ID
```

### Quality String (Phred+33)

The 4th line encodes per-base Q-scores:

```python
def parse_fastq_record(lines: list[str]) -> dict:
    if len(lines) != 4:
        raise ValueError("FASTQ record must have 4 lines")
    return {
        "id": lines[0].lstrip("@"),
        "seq": lines[1],
        "plus": lines[2],
        "qual": [ord(c) - 33 for c in lines[3].strip()],
    }

def format_fastq(record: dict) -> str:
    qual = "".join(chr(q + 33) for q in record["qual"])
    return f"@{record['id']}\n{record['seq']}\n+\n{qual}\n"
```

### FASTQ in the Wild

```bash
# Count reads
zcat sample.fastq.gz | wc -l
# Divide by 4 to get read count

# View first read
zcat sample.fastq.gz | head -4

# Filter reads by minimum length
zcat sample.fastq.gz | awk 'NR%4==2{len=length} NR%4==1{hdr=$0} \
    len>=50{if(NR%4==1)print hdr; print}'

# Convert FASTQ to FASTA (drop quality)
zcat sample.fastq.gz | paste - - - - | awk -F'\t' \
    '{print ">" $1 "\n" $2}' | head -10
```

---

## 3. Compression

Genomics files are always compressed:

```bash
# Uncompressed vs compressed
sampledata.fastq        = 4.5 GB   (never store like this)
sampledata.fastq.gz     = 1.2 GB   (gzip, standard)
sampledata.fastq.bz2    = 900 MB   (better compression, slower)
```

### gzip vs bgzip

Standard `gzip` can't be indexed for random access. **bgzip** (block gzip) can:

```bash
# Compress with bgzip (samtools-compatible)
bgzip -c sample.fastq > sample.fastq.gz

# Create tabix index for random access
tabix -p vcf variants.vcf.gz   # for VCF
samtools index aligned.bam     # for BAM (uses BAI, not tabix)
```

**Rule:** Use `bgzip` (not `gzip`) for any file that needs random access.

---

## 4. Common FASTQ Problems

### Mislabeled Pairs

If R1 and R2 are swapped, alignment stats look terrible:

```bash
# Check read 1 / read 2 in FASTQ
zcat sample_R1.fastq.gz | head -1 | grep -c "1$"   # should end with /1
zcat sample_R2.fastq.gz | head -1 | grep -c "2$"   # should end with /2
```

### Interleaved FASTQ

Some workflows use interleaved FASTQ (R1 and R2 alternating):

```
@read/1 ACTG
+
IIII
@read/2 ACTG
+
IIII
```

### Unmatched Read Counts (R1 vs R2)

Both files must have the same number of reads:

```bash
# Compare read counts
echo "R1: $(zcat sample_R1.fastq.gz | wc -l) / 4 = $(zcat sample_R1.fastq.gz | wc -l | awk '{print $1/4}')"
echo "R2: $(zcat sample_R2.fastq.gz | wc -l) / 4 = $(zcat sample_R2.fastq.gz | wc -l | awk '{print $1/4}')"
```

---

## 5. Writing a FASTQ Parser in C# (Portfolio Context)

```csharp
public class FastqRecord
{
    public string Id { get; set; } = "";
    public string Sequence { get; set; } = "";
    public string Quality { get; set; } = "";

    public double MeanQuality() =>
        Quality.Average(c => c - 33);

    public bool IsHighQuality(double threshold = 30) =>
        Quality.All(c => c - 33 >= threshold);
}

public class FastqReader : IDisposable
{
    private readonly GZipStream? _gzip;
    private readonly StreamReader _reader;

    public FastqReader(string path)
    {
        var stream = File.OpenRead(path);
        if (path.EndsWith(".gz"))
        {
            _gzip = new GZipStream(stream, CompressionMode.Decompress);
            _reader = new StreamReader(_gzip);
        }
        else
        {
            _reader = new StreamReader(stream);
        }
    }

    public FastqRecord? ReadNext()
    {
        var id = _reader.ReadLine();
        if (id == null) return null;
        var seq = _reader.ReadLine()!;
        var plus = _reader.ReadLine()!;
        var qual = _reader.ReadLine()!;
        return new FastqRecord
        {
            Id = id.TrimStart('@'),
            Sequence = seq,
            Quality = qual
        };
    }

    public void Dispose() => _reader.Dispose();
}
```

---

## 6. File Size Estimation

```python
def estimate_fastq_size(
    num_reads: int,
    read_length: int,
    gzipped: bool = True,
) -> float:
    """Estimate FASTQ file size in GB."""
    bytes_per_read = (
        len(f"@read\n") +         # header
        read_length +             # sequence
        len("+\n") +              # separator
        read_length +             # quality
        len("\n")                 # newline
    )
    total = num_reads * bytes_per_read
    if gzipped:
        total *= 0.3  # ~3:1 compression for DNA
    return total / (1024**3)

# 30× human WGS: ~600M reads × 150 bp
print(f"{estimate_fastq_size(600_000_000, 150, True):.1f} GB")
# Output: ~250 GB
```

---

## Exercises

1. Write a Python or C# function that reads a FASTA file and returns a dictionary of sequence ID → sequence.
2. Write a function that converts a FASTQ file to FASTA (drop quality scores).
3. Given a FASTQ file, count the number of reads with mean Q-score below 20 and report the percentage.
4. Parse a FASTQ file and report the distribution of read lengths. Why might some reads be shorter than expected?
5. Download a test FASTA sequence for TP53 and use `samtools faidx` to extract a specific region.

---

## Key Terms

| Term | Definition |
|------|------------|
| FASTA | Text format for reference sequences (one header `>`, then sequence) |
| FASTQ | Text format for sequencing reads (4 lines per read: header, seq, +, qual) |
| Quality encoding | Phred+33 ASCII encoding of per-base Q-scores |
| bgzip | Block gzip compression (supports random access indexing) |
| fai | FASTA index file for random region extraction |
| Read ID | The unique identifier string in the FASTQ header |

---

## Next Steps

→ `12-sam-bam-cram.md` — Aligned read formats (SAM/BAM/CRAM)  
→ `13-vcf-bcf.md` — Variant file formats  
→ `14-samtools-deep-dive.md` — Processing these files  

---

*"FASTQ is raw sensor data. Treat it as irreplaceable."*
