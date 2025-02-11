FROM ghcr.io/ledgerhq/ledger-app-builder/ledger-app-builder-lite:latest AS LITE_BUILDER

FROM ubuntu:22.04
ENV LANG C.UTF-8

ARG DEBIAN_FRONTEND=noninteractive

ARG LLVM_VERSION=12

RUN apt-get update && apt-get upgrade -qy
RUN apt-get install -qy \
        clang-$LLVM_VERSION \
        cmake \
        curl \
        git \
        lld-$LLVM_VERSION \
        make \
        protobuf-compiler \
        python-is-python3 \
        python3 \
        python3-pip && \
    apt-get autoclean -y && \
    apt-get autoremove -y && \
    apt-get clean

# Create generic clang & lld symbolic links to their installed version
RUN cd /usr/bin && \
    find . -name "*-"$LLVM_VERSION | sed "s/^\(.*\)\(-"$LLVM_VERSION"\)$/ln -s \1\2 \1/" | sh

# ARM Embedded Toolchain
# Integrity is checked using the MD5 checksum provided by ARM at https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads
RUN curl -sSfL -o arm-toolchain.tar.bz2 "https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/10.3-2021.10/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2" && \
    echo 2383e4eb4ea23f248d33adc70dc3227e arm-toolchain.tar.bz2 > /tmp/arm-toolchain.md5 && \
    md5sum --check /tmp/arm-toolchain.md5 && rm /tmp/arm-toolchain.md5 && \
    tar xf arm-toolchain.tar.bz2 -C /opt && \
    rm arm-toolchain.tar.bz2

# Adding GCC to PATH and defining rustup/cargo home directories
ENV PATH=/opt/gcc-arm-none-eabi-10.3-2021.10/bin:$PATH \
    RUSTUP_HOME=/opt/rustup \
    CARGO_HOME=/opt/.cargo

# Adding cargo binaries to PATH
ENV PATH=${CARGO_HOME}/bin:${PATH}

# Installing rustup to manage rust toolchains and setting the default toolchain to the 1.67.0 stable channel
RUN curl https://sh.rustup.rs -sSf | \
    sh -s -- --default-toolchain 1.67.0 -y

# Installing nightly channel version 1.67.0 as well.
RUN rustup install nightly-2022-12-02

# Adding ARMV6M target to the installed toolchains
RUN rustup target add thumbv6m-none-eabi --toolchain 1.67.0
RUN rustup target add thumbv6m-none-eabi --toolchain nightly-2022-12-02

# Adding rust-src component to nightly channel
RUN rustup component add rust-src --toolchain nightly-2022-12-02

# Python package to load app onto device
RUN pip3 install ledgerblue

ENV NANOS_SDK=/opt/nanos-secure-sdk
ENV NANOX_SDK=/opt/nanox-secure-sdk
ENV NANOSP_SDK=/opt/nanosplus-secure-sdk
ENV STAX_SDK=/opt/stax-secure-sdk

# Copy the SDKs from the lite container
COPY --from=LITE_BUILDER /opt/ /opt/

# Default SDK
ENV BOLOS_SDK=${NANOS_SDK}

WORKDIR /app

# Work around the git security to be able to get informations from repositories
# even if the container is not run with root UID/GID
RUN git config --system --add safe.directory "*"

CMD ["/usr/bin/env", "bash"]
