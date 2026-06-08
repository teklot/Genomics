# 16 — Nextflow DSL2: Pipeline Automation

**Prerequisites:** `10-quality-control.md`, `14-samtools-deep-dive.md`, `15-bcftools-deep-dive.md`

**See also:** `17-containers-bioinformatics.md`, `18-aws-genomics.md`, `19-pipeline-integration.md`

---

## The Big Idea

**Nextflow** is a workflow engine for scientific pipelines. It's like GitHub Actions for bioinformatics — but designed for massive data volumes, HPC/cloud executors, and reproducible science.

```
Software analogy:
  GitHub Actions + Docker + Airflow × genomics
```

---

## 1. What Nextflow Solves

| Problem | Nextflow Solution |
|---------|------------------|
| Reproducibility | Containerized processes (Docker/Singularity) |
| Data locality | Automated staging to HPC/cloud storage |
| Parallelism | Processes run concurrently when dependencies allow |
| Resumability | Cached intermediate results (resume flag) |
| Portability | Same pipeline runs on laptop, cluster, or cloud |
| Complexity | DSL2 module system for reusable components |

---

## 2. Core Concepts

```
Pipeline = Workflow definition (composed of reusable Modules)
Module   = A single process (e.g., "run FastQC")
Process  = A task: input → script → output
Channel  = Dataflow variable (queues, values, files)
Operator = Transform channels (map, filter, group, branch)
Executor = Where processes run (local, SGE, AWS Batch)
```

### Hello World in Nextflow

```groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

process sayHello {
    input:
    val x

    output:
    stdout

    script:
    """
    echo "Hello, $x!"
    """
}

workflow {
    Channel.of("World", "Genomics", "Nextflow") | sayHello | view { "Result: $it" }
}
```

```bash
# Run
nextflow run hello.nf

# Resume (if it failed partway, continue from cache)
nextflow run hello.nf -resume

# With a different config
nextflow run hello.nf -c production.config
```

---

## 3. DSL2 Module System

### Main Workflow (`main.nf`)

```groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

// Include modules
include { FASTQC } from './modules/fastqc'
include { TRIMGALORE } from './modules/trimgalore'
include { BWA_ALIGN } from './modules/bwa.nf'

// Parameters
params.reads = "data/*_R{1,2}.fastq.gz"
params.outdir = "./results"
params.genome = "hg38.fa"

workflow {
    // Read channel from parameter
    read_pairs = Channel.fromFilePairs(params.reads, size: 2)

    // Run QC
    FASTQC(read_pairs)

    // Trim
    TRIMGALORE(read_pairs)
    trimmed = TRIMGALORE.out.reads

    // Align
    BWA_ALIGN(trimmed, file(params.genome))

    // Publish results
    BWA_ALIGN.out.bam | collect | set { all_bams }
}
```

### Module (`modules/bwa.nf`)

```groovy
process BWA_ALIGN {
    tag "${sample_id}"
    label 'bioinfo_8cpu'
    publishDir "${params.outdir}/bam", mode: 'copy'

    input:
    tuple val(sample_id), path(reads)

    output:
    tuple val(sample_id), path("${sample_id}.sorted.bam"), emit: bam
    path("${sample_id}.sorted.bam.bai"), emit: bai

    script:
    """
    bwa mem -t ${task.cpus} ${genome} ${reads[0]} ${reads[1]} | \
        samtools sort -@ 2 -o ${sample_id}.sorted.bam
    samtools index ${sample_id}.sorted.bam
    """
}
```

### Process Directives

```groovy
process EXAMPLE {
    // Resource management
    cpus 8                            // CPUs for this process
    memory '32 GB'                    // Memory allocation
    time '2.h'                        // Time limit

    // Execution
    label 'big_mem'                   // Reference a label in config
    accelerator 1, type: 'gpu'        // GPU support
    queue 'high-mem'                  // Specific queue

    // I/O
    publishDir '/output', mode: 'link'    // Copy results here
    storeDir '/cache'                     // Persistent cache

    // Container
    container 'biocontainers/bwa:v1.0'    // Single container
    conda 'env.yml'                       // Or conda environment

    // Execution control
    maxRetries 3                      // Retry on failure
    maxErrors '-1'                    // Unlimited errors in a batch
    errorStrategy 'retry'             // What to do on error
}
```

---

## 4. Channels — The Dataflow Variables

### Channel Types

```groovy
// Value channel (single value, reusable)
genome = Channel.value(file("/data/hg38.fa"))
ref_dict = Channel.value("/data/hg38.dict")

// Queue channel (consumed once)
reads = Channel.fromPath("/data/*.fastq.gz")
pairs = Channel.fromFilePairs("/data/*_R{1,2}.fastq.gz")

// Creating channels
Channel.of(1, 2, 3, 4, 5)
Channel.fromList(["a", "b", "c"])
Channel.fromPath(params.reads)
Channel.watchPath("/incoming/*.fastq", "create")  // Real-time monitoring
Channel.empty()                                     // Empty channel
```

### Channel Operators

```groovy
// Transform
reads | map { it -> tuple(it.baseName, it) }       // Map
reads | filter { it.size() > 1000 }                // Filter
reads | flatten                                     // Flatten nested
reads | groupTuple                                  // Group by key
reads | collect                                     // Collect all

// Combine
ch1 | mix(ch2)                                      // Merge channels
ch1 | join(ch2)                                     // Join on key
ch1 | combine(ch2)                                  // Cartesian product
ch1 | phase                                         // Phase channels

// Branch
reads | branch {
    small: it.size() < 100
    medium: it.size() < 1000
    large: true
}

// Multi-map
reads | multiMap {
    sample: it
    stats: it + ".stats"
    log: it + ".log"
}
```

---

## 5. Parameters & Configuration

### `nextflow.config`

```groovy
// Global config file
params {
    reads = "data/*_R{1,2}.fastq.gz"
    outdir = "./results"
    genome = "/ref/hg38.fa"
}

profiles {
    standard {
        process.executor = 'local'
        process.cpus = 2
    }

    cluster {
        process.executor = 'sge'
        process.queue = 'long'
        process.memory = '16 GB'
    }

    aws {
        process.executor = 'awsbatch'
        process.queue = 'genomics-queue'
        aws.region = 'us-east-1'
        aws.batch.cliPath = '/home/ec2-user/miniconda/bin/aws'
    }
}
```

```bash
# Use profiles
nextflow run main.nf -profile standard
nextflow run main.nf -profile aws
nextflow run main.nf -profile cluster

# Override params inline
nextflow run main.nf --reads "/data/new/*.fastq.gz" --outdir "/results/new"
```

---

## 6. nf-core — Community Pipelines

[nf-core](https://nf-co.re/) provides production-ready, peer-reviewed Nextflow pipelines:

```bash
# Popular nf-core pipelines
nf-core list

# Run Sarek (germline/somatic WGS)
nextflow run nf-core/sarek -profile test,docker

# Run rnaseq
nextflow run nf-core/rnaseq -profile test,docker

# Run viralrecon
nextflow run nf-core/viralrecon -profile test,docker
```

**Why nf-core matters:**
- Built by experts, peer-reviewed
- Containerized (Docker/Singularity)
- Cloud-ready (AWS, Azure, GCP)
- Standardized input/output formats
- Active community support

---

## 7. Resources & Labels

### Resource Profiles

```groovy
// Define resource labels in config
process {
    withLabel: 'bioinfo_8cpu' {
        cpus = 8
        memory = '32 GB'
        time = '4.h'
    }
    withLabel: 'bioinfo_16cpu' {
        cpus = 16
        memory = '64 GB'
        time = '8.h'
    }
    withName: 'FASTQC' {
        cpus = 2
        memory = '4 GB'
    }
}
```

---

## 8. Real Pipeline: WGS Germline

### `main.nf`

```groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

params.reads = "data/*_R{1,2}.fastq.gz"
params.outdir = "./results"
params.genome = "/ref/hg38.fa"
params.dbsnp = "/ref/dbsnp.vcf.gz"
params.known_indels = "/ref/Mills_and_1000G_gold_standard.indels.hg38.vcf.gz"

include { FASTQC } from './modules/fastqc'
include { FASTP } from './modules/fastp'
include { BWA_ALIGN } from './modules/bwa'
include { MARKDUPLICATES } from './modules/markduplicates'
include { BQSR } from './modules/bqsr'
include { HAPLOTYPECALLER } from './modules/haplotypecaller'
include { VCF_POSTPROCESS } from './modules/vcf_postprocess'

workflow {
    read_pairs = Channel.fromFilePairs(params.reads, size: 2)

    // QC
    FASTQC(read_pairs)

    // Trim
    FASTP(read_pairs)
    trimmed = FASTP.out.reads

    // Align
    BWA_ALIGN(trimmed, file(params.genome))
    aligned = BWA_ALIGN.out.bam

    // Post-alignment
    MARKDUPLICATES(aligned)
    deduped = MARKDUPLICATES.out.bam

    // BQSR
    BQSR(deduped, file(params.genome), file(params.dbsnp), file(params.known_indels))
    recalibrated = BQSR.out.bam

    // Call variants
    HAPLOTYPECALLER(recalibrated, file(params.genome))
    raw_vcf = HAPLOTYPECALLER.out.vcf

    // Post-process
    VCF_POSTPROCESS(raw_vcf, file(params.genome))
}
```

---

## 9. Running Pipelines

```bash
# Local execution (test)
nextflow run main.nf -profile test -resume

# With containers
nextflow run main.nf -profile docker

# With SLURM cluster
nextflow run main.nf -profile slurm

# On AWS Batch
nextflow run main.nf -profile aws -bucket-dir s3://my-bucket/work/

# Trace execution
nextflow run main.nf -with-trace

# Timeline visualization
nextflow run main.nf -with-timeline

# Report
nextflow run main.nf -with-report

# Email notification (when using -N)
nextflow run main.nf -N user@example.com
```

---

## 10. Caching and Resumability

Nextflow caches every task by inputs + code hash:

```bash
# First run (takes 2 hours)
nextflow run main.nf

# Second run (same inputs) — everything cached
nextflow run main.nf -resume

# Change one input file — only affected processes rerun
nextflow run main.nf -resume

# Force rerun of specific process (when changing module code)
# Just change the module file, Nextflow detects the hash change
```

**This is one of the most powerful features:** interim pipeline failures only require re-running, not restarting.

---

## Exercises

1. Write a Nextflow pipeline with two processes: `FASTQC` (input: FASTQ, output: HTML) and `VIEW_RESULTS` (input: HTML, output: logs "QC done for X").
2. Run the nf-core/sarek pipeline on test data. What steps does it execute?
3. Convert the BWA alignment command from `14-samtools-deep-dive.md` into a Nextflow module.
4. Create a `nextflow.config` with profiles for local, docker, and aws execution.
5. Research: What is the difference between `Channel.fromPath` and `Channel.fromFilePairs`? When would you use each?

---

## Key Terms

| Term | Definition |
|------|------------|
| Nextflow | Workflow engine for scientific pipelines |
| DSL2 | Domain-specific language version 2 (module system) |
| Process | A single computational task |
| Channel | Dataflow variable connecting processes |
| Operator | Channel transformation (map, filter, join, etc.) |
| Executor | Runtime backend (local, SGE, AWS Batch, Kubernetes) |
| Profile | Named configuration for different execution environments |
| nf-core | Community collection of production-ready pipelines |
| Resume | Use cached results to skip completed tasks |

---

## Next Steps

→ `17-containers-bioinformatics.md` — Containerizing tools  
→ `18-aws-genomics.md` — Cloud infrastructure for genomics  
→ `19-pipeline-integration.md` — End-to-end pipeline  

---

*"Nextflow makes pipelines reproducible, portable, and parallel. It's the git + Docker of bioinformatics workflows."*
