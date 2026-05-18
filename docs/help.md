taf-merqury 1.3-r1

Usage:
  taf-merqury [TAF-APP-OPTION]
  taf-merqury merqury.sh [MERQURY-ARGS...]
  taf-merqury meryl [MERYL-ARGS...]
  taf-merqury best_k.sh <GENOME_SIZE> [ERROR_RATE]
  taf-merqury qv.sh <READS.meryl> <ASM.fa> [ASM2.fa] <OUT>
  taf-merqury -- [MERQURY-ARGS...]
  taf-merqury <COMMAND> [COMMAND-ARGS...]

TAF app options:
  -h, --help       Show this help text
  -v, --version    Show package and command version
  --compile        Print generated shell code instead of running it
  --               Stop parsing TAFFISH wrapper options

Upstream help:
  taf-merqury merqury.sh
  taf-merqury --
  taf-merqury plot_spectra_cn.R --help

Recommended examples:
  taf-merqury meryl k=21 memory=64 count reads.fastq.gz output read-db.meryl
  taf-merqury merqury.sh read-db.meryl asm1.fasta out_prefix
  taf-merqury merqury.sh read-db.meryl asm1.fasta asm2.fasta out_prefix
  taf-merqury merqury.sh read-db.meryl mat.hapmer.meryl pat.hapmer.meryl asm1.fasta out_prefix
  taf-merqury qv.sh read-db.meryl asm1.fasta out_prefix
  taf-merqury best_k.sh 3100000000

Main workflow:
  merqury.sh <read-db.meryl> [<mat.meryl> <pat.meryl>] <asm1.fasta> [asm2.fasta] <out>

Common outputs:
  <out>.qv
  <out>.completeness.stats
  <out>.<asm>.spectra-cn.*.png
  <out>.spectra-asm.*.png
  <asm>_only.bed
  <asm>_only.wig
  logs/

Notes:
  - This command runs Merqury inside the TAFFISH container image.
  - The default command is merqury.sh. Option-leading default-command calls can
    use "taf-merqury -- ..."; command-mode examples should name the executable
    explicitly, such as "taf-merqury merqury.sh ...".
  - The image sets MERQURY=/opt/merqury and exposes Merqury root, build, eval,
    plot, trio, and util directories on PATH.
  - Meryl databases must be directories named with the .meryl suffix.
  - Merqury may generate asm.meryl directories in the current working directory.
    Avoid output prefixes or assembly basenames that collide with existing files.
  - Assembly-only BED/WIG outputs can be empty when every assembly k-mer is also
    present in the read database.
  - Meryl uses OpenMP and can use many CPUs. Set OMP_NUM_THREADS through your
    shell or container backend when you need a fixed thread count.
  - This package tracks Merqury v1.3 and bundles Meryl 1.4.1, matching current
    upstream dependency guidance. Unreleased Merqury master script behavior for
    Verkko homopolymer-compressed hapmers is not claimed by this 1.3-r1 package.
  - This app is linux/amd64 only. Docker and Podman runs declare
    --platform linux/amd64; arm64 hosts use container emulation.
  - Slurm _submit_*.sh scripts are included as upstream files, but sbatch is not
    bundled or configured.
  - Optional BigWig helpers that call UCSC wigToBigWig are included, but
    wigToBigWig is not bundled. Core Merqury v1.3 writes .wig files.
  - Large example data and pre-built public Meryl databases are not bundled.

Container:
  image: ghcr.io/taffish/merqury:1.3-r1
  bundled meryl: 1.4.1
  supported backends: apptainer, podman, docker
  supported platforms: linux/amd64

Upstream:
  project: Merqury
  source:  https://github.com/marbl/merqury
  release: https://github.com/marbl/merqury/releases/tag/v1.3
  bundled meryl: https://github.com/marbl/meryl/releases/tag/v1.4.1
  license: Public Domain / United States Government Work notice
  citation: Rhie et al. 2020, doi:10.1186/s13059-020-02134-9, PMID:32928274
