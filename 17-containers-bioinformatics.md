# 17 — Containers for Bioinformatics

**Prerequisites:** Basic Docker knowledge, Linux CLI

**See also:** `16-nextflow-dsl2.md`, `18-aws-genomics.md`, `19-pipeline-integration.md`

---

## The Big Idea

Containers solve the fundamental reproducibility problem: **"It worked on my machine"** is not acceptable for a clinical or research pipeline.

```
Without containers:
  Tool v1.2 on my machine vs v2.0 on yours → different results

With containers:
  Tool v1.2 in a container → same result everywhere
```

---

## 1. Why Containers for Genomics

| Problem | Container Solution |
|---------|-------------------|
| Tool has 30+ dependencies | One `Dockerfile` captures all dependencies |
| Tool only works on Ubuntu 18.04 | Container supplies the OS |
| GCC/OpenMPI version mismatch | Pinned in the image |
| "Upgrade broke my pipeline" | Images are immutable versions |
| Cluster doesn't have the tool | Pull the image |
| Reviewer can't reproduce results | Container + pipeline = reproducible |

### The BioContainers Project

[BioContainers](https://biocontainers.pro/) provides pre-built Docker/Singularity images for >20,000 bioinformatics tools:

```bash
# Pull a BioContainer
docker pull biocontainers/fastqc:v0.11.9_cv8
docker pull biocontainers/bwa:v0.7.17
docker pull biocontainers/samtools:v1.15.1
docker pull biocontainers/bcftools:v1.15.1

# Use directly
docker run biocontainers/fastqc:v0.11.9_cv8 fastqc --help
```

---

## 2. Docker for Bioinformatics

### Example Dockerfile for a Pipeline Tool

```dockerfile
FROM ubuntu:22.04

LABEL maintainer="you@example.com"
LABEL description="WGS Germline Pipeline Container"

# Avoid apt-get interactive prompts
ENV DEBIAN_FRONTEND=noninteractive

# System dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    wget \
    curl \
    git \
    libncurses5-dev \
    libbz2-dev \
    liblzma-dev \
    zlib1g-dev \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install BWA
RUN wget https://github.com/lh3/bwa/releases/download/v0.7.17/bwa-0.7.17.tar.bz2 && \
    tar -xjf bwa-0.7.17.tar.bz2 && \
    cd bwa-0.7.17 && make && \
    cp bwa /usr/local/bin/

# Install SAMtools
RUN wget https://github.com/samtools/samtools/releases/download/1.17/samtools-1.17.tar.bz2 && \
    tar -xjf samtools-1.17.tar.bz2 && \
    cd samtools-1.17 && ./configure && make && make install

# Install BCFtools
RUN wget https://github.com/samtools/bcftools/releases/download/1.17/bcftools-1.17.tar.bz2 && \
    tar -xjf bcftools-1.17.tar.bz2 && \
    cd bcftools-1.17 && ./configure && make && make install

# Install FastQC
RUN wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.12.1.zip && \
    unzip fastqc_v0.12.1.zip && \
    chmod +x FastQC/fastqc && \
    ln -s /FastQC/fastqc /usr/local/bin/fastqc

# Working directory
WORKDIR /data

# Default command
CMD ["--help"]
```

### Build and Run

```bash
# Build
docker build -t genomics-pipeline:1.0 .

# Run a specific tool
docker run --rm -v $(pwd)/data:/data genomics-pipeline:1.0 bwa mem /ref/hg38.fa R1.fastq.gz R2.fastq.gz

# Interactive
docker run --rm -it -v $(pwd)/data:/data genomics-pipeline:1.0 /bin/bash
```

### Using Multi-Stage Builds

```mermaid
%%{init:{'theme':'base','themeVariables':{'primaryColor':'#e8f4f8','lineColor':'#2b6cb0','fontFamily':'Consolas'}}}%%
flowchart LR
    subgraph Build
        A[ubuntu:22.04 AS builder] --> B[apt-get build-essential]
        B --> C[Download & compile BWA]
        C --> D[bwa binary]
    end
    subgraph Runtime
        E[ubuntu:22.04] --> F[COPY --from=builder bwa]
        F --> G[/usr/local/bin/bwa]
    end
    D -.-> F
    style A fill:#c05621,color:#fff
    style E fill:#276749,color:#fff
    style D fill:#d69e2e
    style G fill:#38a169,color:#fff
```

```dockerfile
# Build stage
FROM ubuntu:22.04 AS builder
RUN apt-get update && apt-get install -y build-essential wget
RUN wget https://github.com/lh3/bwa/releases/download/v0.7.17/bwa-0.7.17.tar.bz2 && \
    tar -xjf bwa-0.7.17.tar.bz2 && \
    cd bwa-0.7.17 && make

# Runtime stage (much smaller)
FROM ubuntu:22.04
COPY --from=builder /bwa-0.7.17/bwa /usr/local/bin/
# No build tools needed in final image → smaller, more secure
```

---

## 3. Singularity/Apptainer for HPC

Most HPC clusters don't allow Docker (root requirement). **Singularity** (now Apptainer) is the HPC standard:

```bash
# Convert Docker image to Singularity
singularity pull docker://biocontainers/fastqc:v0.11.9_cv8
# Produces: fastqc_v0.11.9_cv8.sif

# Build from a definition file
singularity build my_pipeline.sif my_pipeline.def

# Run
singularity exec my_pipeline.sif bwa mem -t 8 /ref/hg38.fa reads.fastq.gz
singularity shell my_pipeline.sif  # interactive shell

# Bind mount directories
singularity exec --bind /data:/data my_pipeline.sif samtools view /data/sample.bam
```

### Singularity Definition File

```singularity
Bootstrap: docker
From: ubuntu:22.04

%post
    apt-get update
    apt-get install -y samtools bcftools bwa
    apt-get clean

%runscript
    echo "Genomics Pipeline Container"
    exec "$@"
```

### Why HPC prefers Singularity

- No root daemon (users don't need root)
- Integrates with HPC schedulers (SLURM, PBS)
- Filesystem integration (bind mounts are natural)
- No image layer caching (smaller footprint on shared FS)

---

## 4. Container Registries

```bash
# Docker Hub (public)
docker push myorg/genomics-pipeline:1.0

# GitHub Container Registry
docker push ghcr.io/myorg/genomics-pipeline:1.0

# AWS ECR (private)
aws ecr get-login-password --region us-east-1 | \
    docker login --username AWS --password-stdin ACCOUNT.dkr.ecr.us-east-1.amazonaws.com
docker push ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/genomics-pipeline:1.0

# Quay.io (Red Hat, common for bio containers)
docker push quay.io/myorg/genomics-pipeline:1.0
```

---

## 5. CI/CD for Container Images

### GitHub Actions Workflow

```yaml
name: Build and Push Container

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:latest
            ghcr.io/${{ github.repository }}:${{ github.sha }}
```

---

## 6. Nextflow + Containers

Nextflow integrates containers natively:

```nextflow.config
docker {
    enabled = true
}

process {
    container = 'biocontainers/bwa:v0.7.17'
}

// Override per process
process BWA_ALIGN {
    container = 'biocontainers/bwa:v0.7.17'
}

process FASTQC {
    container = 'biocontainers/fastqc:v0.11.9_cv8'
}
```

```bash
# Nextflow auto-pulls containers
nextflow run main.nf -profile docker

# For Singularity
nextflow run main.nf -profile singularity

# Conda (alternative)
nextflow run main.nf -profile conda
```

---

## 7. Example: Complete Containerized Pipeline

```dockerfile
# Dockerfile.pipeline
FROM ubuntu:22.04 AS base

# Install common tools
RUN apt-get update && apt-get install -y \
    wget curl unzip bzip2 \
    build-essential gcc \
    libncurses5-dev zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

# BWA
FROM base AS bwa_build
RUN wget -q https://github.com/lh3/bwa/releases/download/v0.7.17/bwa-0.7.17.tar.bz2 && \
    tar -xjf bwa-0.7.17.tar.bz2 && cd bwa-0.7.17 && make

# SAMtools
FROM base AS samtools_build
RUN wget -q https://github.com/samtools/samtools/releases/download/1.17/samtools-1.17.tar.bz2 && \
    tar -xjf samtools-1.17.tar.bz2 && \
    cd samtools-1.17 && ./configure --without-curses && make && make install

# Final
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    libncurses5 liblzma5 libbz2-1.0 perl \
    && rm -rf /var/lib/apt/lists/*

COPY --from=bwa_build /bwa-0.7.17/bwa /usr/local/bin/
COPY --from=samtools_build /usr/local/bin/samtools /usr/local/bin/
COPY --from=samtools_build /usr/local/bin/bcftools /usr/local/bin/

# Add pipeline scripts
COPY scripts/ /pipeline/

WORKDIR /data
ENTRYPOINT ["/pipeline/run.sh"]
```

---

## 8. Container Best Practices

```
✓ Use specific tags (not "latest")
✓ Multi-stage builds for smaller images
✓ Pin dependency versions
✓ Scan for vulnerabilities (trivy, grype)
✓ Test container locally before pushing
✓ Use BioContainers when possible
✓ Document the container in a README
```

```bash
# Security scan
trivy image biocontainers/fastqc:v0.11.9_cv8

# Check container size
docker images biocontainers/bwa
```

---

## Exercises

1. Write a Dockerfile that packages FastQC, BWA, SAMtools, and BCFtools. The final image should be under 500 MB.
2. Convert the Dockerfile to a Singularity definition file.
3. Set up a GitHub Actions workflow that builds and pushes the container to GitHub Container Registry on tags.
4. Run a Nextflow pipeline using containers instead of local tools. Compare execution time.
5. Compare **Docker** and **Singularity/Apptainer**. Why would an HPC cluster prefer Singularity over Docker? What security concerns does each address?

---

## Key Terms

| Term | Definition |
|------|------------|
| Docker | Container runtime (requires root) |
| Singularity/Apptainer | HPC-safe container runtime (no root) |
| BioContainers | Registry of pre-built bioinformatics containers |
| Multi-stage build | Building in one image, running in another (smaller) |
| Container registry | Storage for container images (Docker Hub, ECR, GHCR) |
| CI/CD | Automated build and test pipeline |
| Reproducibility | Identical results across environments |

---

## Next Steps

→ `18-aws-genomics.md` — Deploying containers on AWS  
→ `19-pipeline-integration.md` — Putting it all together  

---

*"A pipeline without containers is a recipe, not a reproducible method."*
