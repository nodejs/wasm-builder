# Modify these to update to newer versions
FROM node:22-alpine3.19@sha256:3cb4748ed93c45cf4622c3382a5ce063af1fcbc5f3da6d2b669352ebace9f76d
ARG BINARYEN_VERSION=116

ARG UID=1000
ARG GID=1000

# set SOURCE_DATE_EPOCH for reproduciability - https://reproducible-builds.org/docs/source-date-epoch/
ENV SOURCE_DATE_EPOCH: 0

ENV ROOT_DIR /home/node
ENV METADATA_DIR $ROOT_DIR/metadata
RUN mkdir $METADATA_DIR
ENV WORKDIR_DIR $ROOT_DIR/build
RUN mkdir WORKDIR_DIR
WORKDIR $WORKDIR_DIR

# Keep a copy of the Dockerfile used
COPY Dockerfile /home/node/metadata/Dockerfile

# Install required system packages
RUN apk add -U clang lld wasi-sdk
RUN apk info -v >/home/node/metadata/apk-info

# Install BINARYEN 
RUN wget https://github.com/WebAssembly/binaryen/releases/download/version_$BINARYEN_VERSION/binaryen-version_$BINARYEN_VERSION-x86_64-linux.tar.gz && \
    tar -zxvf binaryen-version_$BINARYEN_VERSION-x86_64-linux.tar.gz binaryen-version_$BINARYEN_VERSION/bin/wasm-opt && \
    mv binaryen-version_$BINARYEN_VERSION/bin/wasm-opt ./ && \
    rm binaryen-version_$BINARYEN_VERSION-x86_64-linux.tar.gz && \
    rm -rf binaryen-version_$BINARYEN_VERSION && \
    chmod +x ./wasm-opt
ENV WASM_OPT=$WORKDIR_DIR/wasm-opt
RUN echo "Binaryen-version: $BINARYEN_VERSION" >$METADATA_DIR/binaryen-info
RUN echo "Binaryen-url: https://github.com/WebAssembly/binaryen/releases/download/version_$BINARYEN_VERSION/binaryen-version_$BINARYEN_VERSION-x86_64-linux.tar.gz" >>$METADATA_DIR/binaryen-info
RUN echo "Binaryen-binary: $WASM_OPT" >>$METADATA_DIR/binaryen-info

USER node
