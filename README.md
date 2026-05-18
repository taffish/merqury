# taf-merqury

TAFFISH wrapper for [Merqury](https://github.com/marbl/merqury), a k-mer based
toolkit for reference-free genome assembly evaluation, QV estimation,
completeness analysis, spectra plots, and trio hap-mer phasing assessment.

This package tracks the latest tagged upstream release, Merqury `v1.3`, and
packages it with upstream-recommended Meryl `v1.4.1`, R plotting dependencies,
Java, BEDTools, SAMtools, and common compression helpers in one container.

## Installation

Install from the public TAFFISH Hub index:

```sh
taf update
taf install merqury
```

Install the exact release:

```sh
taf install merqury 1.3-r1
```

For local testing before the app is published to the public index:

```sh
taf install --from .
```

This app is `linux/amd64` only. The Docker and Podman wrapper declares
`--platform linux/amd64`, so arm64 hosts such as Apple Silicon Macs can run it
through container emulation. Native `linux/arm64` is not claimed because Meryl
`v1.4.1` builds x86 SSE code in its utility library.

## Usage

Show TAFFISH app help:

```sh
taf-merqury --help
```

Show the TAFFISH package version:

```sh
taf-merqury --version
```

Show upstream Merqury usage:

```sh
taf-merqury merqury.sh
taf-merqury --
```

Run the standard single-machine Merqury workflow:

```sh
taf-merqury merqury.sh read-db.meryl asm1.fasta out_prefix
```

Run a two-assembly evaluation:

```sh
taf-merqury merqury.sh read-db.meryl asm1.fasta asm2.fasta out_prefix
```

Run trio/hap-mer mode:

```sh
taf-merqury merqury.sh read-db.meryl mat.hapmer.meryl pat.hapmer.meryl asm1.fasta out_prefix
```

Run a two-assembly trio/hap-mer evaluation:

```sh
taf-merqury merqury.sh read-db.meryl mat.hapmer.meryl pat.hapmer.meryl asm1.fasta asm2.fasta out_prefix
```

Build a read k-mer database with bundled Meryl:

```sh
taf-merqury meryl k=21 memory=64 count reads.fastq.gz output read-db.meryl
```

Estimate a good k size:

```sh
taf-merqury best_k.sh 3100000000
```

Run a focused QV-only helper:

```sh
taf-merqury qv.sh read-db.meryl asm1.fasta out_prefix
```

Because this is a command-mode TAFFISH tool, the first non-option argument is
treated as a command available inside the container. The clearest form is:

```sh
taf-merqury merqury.sh ...
taf-merqury meryl ...
taf-merqury spectra-cn.sh ...
taf-merqury plot_spectra_cn.R --help
```

Option-leading calls can use the default command with `--`:

```sh
taf-merqury -- read-db.meryl asm1.fasta out_prefix
```

## Inputs And Outputs

Merqury assumes Meryl databases are directories named with the `.meryl` suffix.
The main workflow accepts:

```text
merqury.sh <read-db.meryl> [<mat.meryl> <pat.meryl>] <asm1.fasta> [asm2.fasta] <out>
```

Typical outputs include:

```text
<out>.qv
<out>.completeness.stats
<out>.<asm>.spectra-cn.*.png
<out>.spectra-asm.*.png
<asm>_only.bed
<asm>_only.wig
logs/
```

When the read k-mer database and assembly are identical, the assembly-only BED
and WIG files can legitimately be empty.

Merqury creates assembly Meryl databases such as `asm1.meryl` in the current
working directory if they do not already exist. Avoid naming assemblies or
hap-mer databases in a way that collides with generated `.meryl` directories.

Meryl uses OpenMP and may use all available CPUs by default. Set
`OMP_NUM_THREADS` through your container backend or shell when you need a fixed
thread count.

## Packaged Commands

The image puts these Merqury directories on `PATH`:

```text
/opt/merqury
/opt/merqury/build
/opt/merqury/eval
/opt/merqury/plot
/opt/merqury/trio
/opt/merqury/util
```

Important bundled commands and scripts include:

```text
merqury.sh
merqury
best_k.sh
meryl
meryl-lookup
qv.sh
spectra-cn.sh
per_seq_qv.sh
hapmers.sh
spectra-hap.sh
hap_blob.sh
phase_block.sh
block_n_stats.sh
switch_error.sh
plot_spectra_cn.R
plot_blob.R
plot_block_N.R
```

## Container

The container image is built from `docker/Dockerfile`. It downloads and verifies:

```text
Merqury v1.3 source archive
Meryl v1.4.1 source archive
Meryl utility submodule source at its v1.4.1 gitlink
```

It builds Meryl from source and installs Merqury's shell, Java, and R scripts.
Runtime dependencies include:

```text
bash, meryl, meryl-lookup, Rscript, argparse, ggplot2, scales,
Java runtime, bedtools, samtools, pigz, gzip, bzip2, xz
```

The image is built and validated for:

```text
linux/amd64
```

Meryl `v1.4.1` is distributed upstream with a Linux amd64 binary, and its source
build currently uses x86 SSE headers. For that reason this TAFFISH image is
published as `linux/amd64` only.

## Boundaries

This package intentionally tracks Merqury `v1.3`, the latest tagged GitHub
release, while bundling Meryl `v1.4.1` because current upstream Merqury
documentation recommends latest Meryl/Merqury and lists Meryl `v1.4.1` as the
dependency. Merqury scripts are still from the `v1.3` tag, so unreleased
Merqury `master` script behavior for Verkko homopolymer-compressed hapmers is
not claimed by this `1.3-r1` package.

The Slurm `_submit_*.sh` wrappers are included as upstream files, but the image
does not bundle or configure Slurm `sbatch`; adapt those scripts to your site
environment if you use them.

The core Merqury `v1.3` workflow writes `.wig` files. The optional BigWig helper
scripts that call UCSC `wigToBigWig` are included, but `wigToBigWig` is not
bundled in this image. Convert `.wig` files with a separate UCSC tools app or
local site installation when needed.

The image does not download large example data or public pre-built Meryl
databases. Fetch those explicitly in your workflow if you want to reproduce the
upstream Arabidopsis or human examples.

## Smoke Test

The TAFFISH smoke metadata checks:

```text
exist: merqury.sh, merqury, best_k.sh, qv.sh, spectra-cn.sh,
       plot_spectra_cn.R, meryl, meryl-lookup, Rscript, java,
       bedtools, samtools, pigz, bash
test:  packaged Merqury version marker reports 1.3
test:  packaged Meryl version marker reports 1.4.1
test:  MERQURY is set to /opt/merqury
test:  Merqury usage, best_k.sh, Meryl help, meryl-lookup help
test:  R packages argparse, ggplot2, scales are loadable
test:  Java, BEDTools, SAMtools, and pigz are callable
test:  a tiny offline FASTA can be counted with Meryl and analyzed by
       merqury.sh, producing QV, completeness, spectra plots, and wig output
```

This validates container wiring and a minimal non-trio Merqury path. It does
not validate large-genome performance, biological accuracy, trio switch-error
interpretation, Slurm submission, BigWig conversion, or external database
download workflows.

## Upstream

- Project: Merqury
- Source: <https://github.com/marbl/merqury>
- Release: <https://github.com/marbl/merqury/releases/tag/v1.3>
- Bundled Meryl: <https://github.com/marbl/meryl/releases/tag/v1.4.1>
- License: Public Domain / United States Government Work notice
- Citation: Rhie et al. 2020, doi:10.1186/s13059-020-02134-9, PMID:32928274

The repository wrapper files follow the same public-domain notice as upstream
Merqury. Bundled Meryl and Debian packages are distributed under their own
upstream terms.
