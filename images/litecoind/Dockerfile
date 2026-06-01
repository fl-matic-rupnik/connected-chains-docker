# syntax=docker/dockerfile:1.3-labs
# test trigger
FROM debian:trixie@sha256:4ae67669760b807c19f23902a3fd7c121a6a70cf2ae709035674b23e712e4d62 as build

ARG VERSION=v0.21.4

ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/London"

# Remove libdb++-dev after upstream fixes the problem with --disable-wallet requiring it
RUN <<-EOF
    set -e
    apt-get -y update && \
    apt-get -y install \
    git \
    build-essential \
    libtool \
    pkg-config \
    curl \
    cmake
EOF

RUN git clone --single-branch --branch "${VERSION}" https://github.com/litecoin-project/litecoin.git /opt/litecoin

WORKDIR /opt/litecoin

# Define PTHREAD_STACK_MIN to avoid build failure with Boost 1.70.0 libraries
# https://github.com/boostorg/thread/pull/297/commits/74fb0a26099bc51d717f5f154b37231ce7df3e98
# CXXFLAGS and CPPFLAGS can be removed if upstream ever updates Boost to 1.73.0 or newer
RUN <<-EOF
    set -e
    make -C /opt/litecoin/depends/ -j $(getconf _NPROCESSORS_ONLN) NO_QT=1 NO_QR=1 NO_WALLET=1 NO_BDB=1 \
        boost_cxxflags="-std=c++11 -fvisibility=hidden -DPTHREAD_STACK_MIN=16384" \
        boost_cppflags="-DPTHREAD_STACK_MIN=16384"
    ./autogen.sh
    ./configure \
        --disable-tests \
        --disable-bench \
        --disable-wallet \
        --without-gui \
        --prefix=/opt/litecoin/depends/x86_64-pc-linux-gnu/ \
        CXXFLAGS="-DPTHREAD_STACK_MIN=16384" \
        CPPFLAGS="-DPTHREAD_STACK_MIN=16384"
    make -j $(getconf _NPROCESSORS_ONLN)
    make install && \
    find /opt/litecoin/depends/x86_64-pc-linux-gnu/bin/ -type f -executable -exec strip -s {} + 2>/dev/null || true
EOF

COPY litecoin.conf /opt/litecoin/.litecoin/litecoin.conf
RUN mkdir -p /opt/litecoin/.litecoin/db

FROM gcr.io/distroless/cc-debian13:nonroot@sha256:e1fd250ce83d94603e9887ec991156a6c26905a6b0001039b7a43699018c0733 as final

ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/London"

COPY --from=build --chown=65532:65532 /opt/litecoin/depends/x86_64-pc-linux-gnu/bin/ /opt/litecoin/bin/
COPY --from=build --chown=65532:65532 /opt/litecoin/.litecoin/ /opt/litecoin/.litecoin/

ENTRYPOINT ["/opt/litecoin/bin/litecoind", "-conf=/opt/litecoin/.litecoin/litecoin.conf", "-datadir=/opt/litecoin/.litecoin/db", "-txindex=1"]
