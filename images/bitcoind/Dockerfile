# syntax=docker/dockerfile:1.3-labs
# test trigger
FROM debian:trixie@sha256:4ae67669760b807c19f23902a3fd7c121a6a70cf2ae709035674b23e712e4d62 as build

ARG VERSION=v30.0

ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/London"

# https://github.com/bitcoin/bitcoin/blob/master/doc/build-unix.md#linux-distribution-specific-instructions
RUN <<-EOF
    set -e
    apt-get -y update && \
    apt-get -y install \
        git \
        build-essential \
        cmake \
        curl
EOF

RUN git clone --single-branch --branch "${VERSION}" https://github.com/bitcoin/bitcoin.git /opt/bitcoin

WORKDIR /opt/bitcoin

RUN <<-EOF
    set -e
    make -C depends/ -j $(getconf _NPROCESSORS_ONLN) NO_QT=1 NO_QR=1 NO_WALLET=1 NO_IPC=1
    cmake -B build \
        --toolchain /opt/bitcoin/depends/x86_64-pc-linux-gnu/toolchain.cmake \
        -DCMAKE_INSTALL_PREFIX=/opt/bitcoin/build/
    cmake --build build -j $(getconf _NPROCESSORS_ONLN) 
    cmake --install build
    find /opt/bitcoin/build/bin -type f -executable -exec strip -s {} + 2>/dev/null || true
EOF

RUN mkdir -p /opt/bitcoin/.bitcoin/db

FROM gcr.io/distroless/cc-debian13:nonroot@sha256:e1fd250ce83d94603e9887ec991156a6c26905a6b0001039b7a43699018c0733 as final

ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/London"

COPY --from=build --chown=65532:65532 /opt/bitcoin/build/bin /opt/bitcoin/bin
COPY --from=build --chown=65532:65532 /opt/bitcoin/.bitcoin/db /opt/bitcoin/.bitcoin/db
COPY --chown=65532:65532 bitcoin.conf /opt/bitcoin/.bitcoin/bitcoin.conf

ENTRYPOINT ["/opt/bitcoin/bin/bitcoind", "-conf=/opt/bitcoin/.bitcoin/bitcoin.conf", "-datadir=/opt/bitcoin/.bitcoin/db", "-txindex=1"]
