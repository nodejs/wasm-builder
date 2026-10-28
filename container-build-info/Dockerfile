# Modify these to update to newer versions
FROM node:24-alpine3.23@sha256:7fddd9ddeae8196abf4a3ef2de34e11f7b1a722119f91f28ddf1e99dcafdf114
ARG BINARYEN_VERSION=128
ARG RUST_VERSION=1.94.0

ARG UID=1000
ARG GID=1000

# set SOURCE_DATE_EPOCH for reproducibility - https://reproducible-builds.org/docs/source-date-epoch/
ENV SOURCE_DATE_EPOCH=0

ENV ROOT_DIR=/home/node
ENV METADATA_DIR=$ROOT_DIR/metadata
RUN mkdir $METADATA_DIR
ENV WORKDIR_DIR=$ROOT_DIR/build
ENV PATH="$ROOT_DIR/.cargo/bin:$PATH"
RUN mkdir $WORKDIR_DIR
WORKDIR $WORKDIR_DIR

# Keep a copy of the Dockerfile used
COPY Dockerfile /home/node/metadata/Dockerfile

###################################
# Install required system packages

# those needed for undici
RUN apk add -U clang lld wasi-sdk

# those needed for cjs-module-lexer
RUN apk add -U clang lld wasi-sdk make

# those needed for amaro
RUN apk add -U curl rust-wasm cmake gcc g++ bash make

# capture the versions of the packages installed
RUN apk info -v >/home/node/metadata/apk-info

###################################
# Non-package installs

# Install Binaryen from the official release archive for the current Linux arch.
# If the expected archive is unavailable for the selected version/arch, fail the build.
RUN CURRENT_ARCH=`arch` && \
    echo "ARCH:" $CURRENT_ARCH && \
    wget https://github.com/WebAssembly/binaryen/releases/download/version_$BINARYEN_VERSION/binaryen-version_$BINARYEN_VERSION-$CURRENT_ARCH-linux.tar.gz || true && \
    if [ -e "./binaryen-version_$BINARYEN_VERSION-$CURRENT_ARCH-linux.tar.gz" ]; then \
      tar -zxvf binaryen-version_$BINARYEN_VERSION-$CURRENT_ARCH-linux.tar.gz binaryen-version_$BINARYEN_VERSION/bin/wasm-opt && \
      mv binaryen-version_$BINARYEN_VERSION/bin/wasm-opt ./ && \
      rm binaryen-version_$BINARYEN_VERSION-$CURRENT_ARCH-linux.tar.gz && \
      rm -rf binaryen-version_$BINARYEN_VERSION && \
      chmod +x ./wasm-opt; \
    fi && \
    if [ ! -e ./wasm-opt ]; then \
      if [ "$CURRENT_ARCH" != "aarch64" ]; then \
        exit 1; \
      else \
        if [ $BINARYEN_VERSION -gt 116 ]; then \
          exit 1; \
        fi; \
      fi; \
    fi

ENV WASM_OPT=$WORKDIR_DIR/wasm-opt
RUN echo "Binaryen-version: $BINARYEN_VERSION" >$METADATA_DIR/binaryen-info
RUN echo "Binaryen-url: https://github.com/WebAssembly/binaryen/releases/download/version_$BINARYEN_VERSION/binaryen-version_$BINARYEN_VERSION-$(arch)-linux.tar.gz" >>$METADATA_DIR/binaryen-info
RUN echo "Binaryen-binary: $WASM_OPT" >>$METADATA_DIR/binaryen-info

RUN chmod -R 777 $METADATA_DIR
USER node

# Install Rust
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs >$METADATA_DIR/install_rust.sh
RUN bash $METADATA_DIR/install_rust.sh -y --quiet
RUN rustup install $RUST_VERSION
RUN rustup default $RUST_VERSION
