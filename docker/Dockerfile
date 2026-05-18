ARG MERQURY_VERSION=1.3
ARG MERQURY_SOURCE_SHA256=b6e3f54a6fe30acfe6f2b15ebc26a103a51980adfc03522e712a2baa5e79131a
ARG MERYL_VERSION=1.4.1
ARG MERYL_SOURCE_SHA256=f365bfc114ee06c3e3ff8a2d458b24bb40bda97ab45e5de99bb298455ae13866
ARG MERYL_UTILITY_COMMIT=99676106a395899543c20d1086927b97bf5f46eb
ARG MERYL_UTILITY_SOURCE_SHA256=9d66cef62d38240ae7bd1e1bbd27aa65f1c4db15a432e4f1821eebac107d7916
ARG DEBIAN_MIRROR=http://deb.debian.org/debian
ARG DEBIAN_SECURITY_MIRROR=http://deb.debian.org/debian-security

FROM debian:12-slim AS meryl-builder

ARG MERQURY_VERSION
ARG MERQURY_SOURCE_SHA256
ARG MERYL_VERSION
ARG MERYL_SOURCE_SHA256
ARG MERYL_UTILITY_COMMIT
ARG MERYL_UTILITY_SOURCE_SHA256
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            build-essential \
            ca-certificates \
            curl \
            perl \
            tar; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/build/meryl

RUN set -eux; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL \
      "https://github.com/marbl/meryl/archive/refs/tags/v${MERYL_VERSION}.tar.gz" \
      -o meryl.tar.gz; \
    echo "${MERYL_SOURCE_SHA256}  meryl.tar.gz" | sha256sum -c -; \
    tar -xzf meryl.tar.gz --strip-components=1; \
    mkdir -p src/utility; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL \
      "https://github.com/marbl/meryl-utility/archive/${MERYL_UTILITY_COMMIT}.tar.gz" \
      -o meryl-utility.tar.gz; \
    echo "${MERYL_UTILITY_SOURCE_SHA256}  meryl-utility.tar.gz" | sha256sum -c -; \
    tar -xzf meryl-utility.tar.gz -C src/utility --strip-components=1; \
    cd src; \
    make -j"$(nproc)"; \
    mkdir -p /opt/meryl/bin /opt/meryl/share/doc/meryl; \
    cp -a ../build/bin/. /opt/meryl/bin/; \
    install -m 0644 ../README.md ../README.licenses /opt/meryl/share/doc/meryl/; \
    printf "%s\n" "${MERYL_VERSION}" > /opt/meryl/share/doc/meryl/VERSION; \
    /opt/meryl/bin/meryl -h 2>&1 | grep -F "COMMANDS:" >/dev/null; \
    /opt/meryl/bin/meryl-lookup 2>&1 | grep -F "usage:" >/dev/null

WORKDIR /tmp/build/merqury

RUN set -eux; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL \
      "https://github.com/marbl/merqury/archive/refs/tags/v${MERQURY_VERSION}.tar.gz" \
      -o merqury.tar.gz; \
    echo "${MERQURY_SOURCE_SHA256}  merqury.tar.gz" | sha256sum -c -; \
    mkdir -p /opt/merqury; \
    tar -xzf merqury.tar.gz --strip-components=1 -C /opt/merqury; \
    find /opt/merqury -type d -exec chmod 0755 {} +; \
    find /opt/merqury -type f -exec chmod 0644 {} +; \
    find /opt/merqury -name "*.sh" -exec chmod 0755 {} +; \
    find /opt/merqury/plot -name "*.R" -exec chmod 0755 {} +

FROM debian:12-slim

ARG MERQURY_VERSION
ARG MERQURY_SOURCE_SHA256
ARG MERYL_VERSION
ARG DEBIAN_MIRROR
ARG DEBIAN_SECURITY_MIRROR

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    sed -i \
      -e "s|http://deb.debian.org/debian-security|${DEBIAN_SECURITY_MIRROR}|g" \
      -e "s|http://deb.debian.org/debian|${DEBIAN_MIRROR}|g" \
      /etc/apt/sources.list.d/debian.sources; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            bash \
            bedtools \
            bzip2 \
            ca-certificates \
            default-jre-headless \
            fontconfig \
            fonts-dejavu-core \
            gzip \
            libgcc-s1 \
            libgomp1 \
            libstdc++6 \
            pigz \
            r-base-core \
            r-cran-argparse \
            r-cran-ggplot2 \
            r-cran-scales \
            samtools \
            xz-utils; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

COPY --from=meryl-builder /opt/meryl /opt/meryl
COPY --from=meryl-builder /opt/merqury /opt/merqury

RUN set -eux; \
    ln -s /opt/merqury/merqury.sh /usr/local/bin/merqury.sh; \
    ln -s /opt/merqury/merqury.sh /usr/local/bin/merqury; \
    ln -s /opt/merqury/best_k.sh /usr/local/bin/best_k.sh; \
    mkdir -p /usr/local/share/doc/merqury/meryl; \
    install -m 0644 /opt/merqury/README.md /opt/merqury/LICENSE /usr/local/share/doc/merqury/; \
    cp -a /opt/meryl/share/doc/meryl/. /usr/local/share/doc/merqury/meryl/; \
    printf "%s\n" "${MERQURY_VERSION}" > /usr/local/share/doc/merqury/VERSION; \
    printf "%s\n" "${MERYL_VERSION}" > /usr/local/share/doc/merqury/MERYL_VERSION

ENV MERQURY=/opt/merqury
ENV PATH=/opt/meryl/bin:/opt/merqury:/opt/merqury/build:/opt/merqury/eval:/opt/merqury/plot:/opt/merqury/trio:/opt/merqury/util:${PATH}

RUN set -eux; \
    test "$(cat /usr/local/share/doc/merqury/VERSION)" = "${MERQURY_VERSION}"; \
    test "$(cat /usr/local/share/doc/merqury/MERYL_VERSION)" = "${MERYL_VERSION}"; \
    test "$MERQURY" = "/opt/merqury"; \
    merqury.sh 2>&1 | grep -F "Usage: merqury.sh" >/dev/null; \
    merqury 2>&1 | grep -F "Usage: merqury.sh" >/dev/null; \
    best_k.sh 1000000 2>&1 | grep -E "^14\\." >/dev/null; \
    meryl -h 2>&1 | grep -F "COMMANDS:" >/dev/null; \
    meryl-lookup 2>&1 | grep -F "usage:" >/dev/null; \
    Rscript -e "library(argparse); library(ggplot2); library(scales)"; \
    plot_spectra_cn.R --help 2>&1 | grep -F "spectra-cn plots" >/dev/null; \
    java -version >/dev/null 2>&1; \
    bedtools --version >/dev/null; \
    samtools --version >/dev/null; \
    pigz --version >/dev/null

RUN set -eux; \
    mkdir -p /tmp/merqury-check; \
    cd /tmp/merqury-check; \
    printf ">toy\nACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGTACGT\n" > asm.fa; \
    meryl k=21 memory=0.1 count asm.fa output read.meryl >/dev/null 2> meryl.log; \
    test -d read.meryl; \
    merqury.sh read.meryl asm.fa out > run.out 2> run.err; \
    test -s out.qv; \
    test -s out.completeness.stats; \
    test -s out.asm.spectra-cn.ln.png; \
    test -s out.spectra-asm.ln.png; \
    test -e asm_only.wig; \
    grep -F "asm_only.wig generated." logs/out.spectra-cn.log >/dev/null; \
    grep -F "Done!" logs/out.spectra-cn.log >/dev/null; \
    cd /; \
    rm -rf /tmp/merqury-check

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=merqury
