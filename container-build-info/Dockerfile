# Modify these to update to newer versions
FROM node:22-alpine3.19@sha256:3fa6e0fa080872f038aa4171597419e5cc463acec7e12214c9e15172bae0b40b
ARG BINARYEN_VERSION=116

ARG UID=1000
ARG GID=1000

# set SOURCE_DATE_EPOCH for reproduciability - https://reproducible-builds.org/docs/source-date-epoch/
ENV SOURCE_DATE_EPOCH=0

ENV ROOT_DIR=/home/node
ENV METADATA_DIR=$ROOT_DIR/metadata
RUN mkdir $METADATA_DIR
ENV WORKDIR_DIR=$ROOT_DIR/build
RUN mkdir WORKDIR_DIR
WORKDIR $WORKDIR_DIR

# Keep a copy of the Dockerfile used
COPY Dockerfile /home/node/metadata/Dockerfile

###################################
# Install required system packages

# those needed for undici
RUN apk add -U clang lld wasi-sdk

# those needed for cjs-module-lexar
RUN apk add -U clang lld wasi-sdk make

# those needed for amaro
RUN apk add -U rust rust-wasm cargo cmake gcc g++ bash make

# capture the versions of the packages installed
RUN apk info -v >/home/node/metadata/apk-info

###################################
# Non package insalls

# Install BINARYEN, there are only linux/arm64 tar.gz's after 116
# If we expect a versoin to exist and we don't get one error out
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
RUN echo "Binaryen-url: https://github.com/WebAssembly/binaryen/releases/download/version_$BINARYEN_VERSION/binaryen-version_$BINARYEN_VERSION-$CURRENT_ARCH-linux.tar.gz" >>$METADATA_DIR/binaryen-info
RUN echo "Binaryen-binary: $WASM_OPT" >>$METADATA_DIR/binaryen-info

USER node
