# Diagrams Directory

All diagrams in this directory are authored in **SVG** format, ensuring high-quality scaling, editability, and native browser rendering.

## Design System ("Slick")

We use a standardized D2 class system to maintain the "Slick" design aesthetic.

### Color Palette
- **Blue:** `#2563eb` / `#3b82f6` (Class: `dna`) — DNA, reference, primary actions
- **Red:** `#dc2626` / `#ef4444` (Class: `mutated`) — Errors, mutations, danger
- **Green:** `#22c55e` / `#15803d` (Class: `protein`) — Protein, success, completed
- **Amber:** `#f59e0b` / `#b45309` (Class: `rna`) — RNA, warnings, in-progress
- **Purple:** `#a855f7` / `#7e22ce` (Class: `regulatory`) — Regulatory, annotation
- **Grey:** `#475569` / `#94a3b8` (Class: `secondary`) — Meta, labels, secondary

## Tools Used

All diagrams are authored as `.svg` source files. These can be rendered to SVG using the D2 CLI or viewed directly in editors with D2 support.

## File Index

| File | Description |
|------|-------------|
| `aws-genomics.svg` | Full AWS architecture with VPC, Batch, S3, EFS |
| `central-dogma.svg` | DNA → RNA → Protein flow with cellular compartments |
| `chromosome-anatomy.svg` | Chromosome structure with p/q arms, centromere, telomeres |
| `coverage-depth.svg` | Visual representation of read stacking over reference |
| `dna-double-helix.svg` | Double helix with base pairing rules |
| `fastq-record.svg` | Annotated FASTQ 4-line record |
| `fastqc-comparison.svg` | Good vs bad FastQC metrics |
| `frameshift-effect.svg` | Reading frame disruption by indel |
| `gene-structure.svg` | Promoter, exons, introns, UTRs |
| `mutation-types.svg` | SNV, indel, structural variants |
| `nanopore-diagram.svg` | Nanopore sequencing with pore and ionic current |
| `paired-end.svg` | Paired-end read layout with insert size |
| `pipeline-architecture.svg` | End-to-end sequencing pipeline |
| `poisson-coverage.svg` | Poisson distribution at 5×, 15×, 30× |
| `quality-distribution.svg` | Per-base quality, good vs bad lane |
| `sam-format.svg` | SAM 11-column format + conversion commands |
| `seed-and-extend.svg` | Alignment seed-and-extend algorithm |
| `sequencing-sbs.svg` | Illumina sequencing-by-synthesis cycle |
| `smith-waterman.svg` | Smith-Waterman DP alignment matrix |
| `splicing.svg` | Pre-mRNA to mature mRNA splicing |
| `transcription.svg` | DNA to RNA with RNA polymerase |
| `translation-ribosome.svg` | Ribosome with A/P/E sites |
| `vcf-format.svg` | VCF header + records + genotype legend |
