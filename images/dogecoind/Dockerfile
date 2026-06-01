# syntax=docker/dockerfile:1.3-labs
# test trigger
FROM debian:trixie@sha256:4ae67669760b807c19f23902a3fd7c121a6a70cf2ae709035674b23e712e4d62 as build

ARG VERSION=v1.14.9

ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/London"

RUN <<-EOF
    set -e
    apt-get -y update && apt-get -y install \
        git \
        build-essential \
        cmake \
        libtool \
        pkg-config \
        curl
EOF

RUN git clone --single-branch --branch "${VERSION}" https://github.com/dogecoin/dogecoin.git /opt/dogecoin

WORKDIR /opt/dogecoin

RUN <<-EOF
    set -e
    make -C /opt/dogecoin/depends/ -j $(getconf _NPROCESSORS_ONLN) NO_QT=1 NO_QR=1 NO_WALLET=1 NO_BDB=1
    ./autogen.sh
    ./configure \
        --disable-wallet \
        --without-gui \
        --disable-tests \
        --disable-bench \
        --prefix=/opt/dogecoin/depends/x86_64-pc-linux-gnu/
    make -j $(getconf _NPROCESSORS_ONLN)
    make install
    find /opt/dogecoin/depends/x86_64-pc-linux-gnu/bin -type f -executable -exec strip -s {} + 2>/dev/null || true
EOF

COPY dogecoin.conf /opt/dogecoin/.dogecoin/dogecoin.conf

RUN <<-EOF
    set -e
    mkdir -p /opt/dogecoin/.dogecoin/db
EOF

FROM gcr.io/distroless/cc-debian13:nonroot@sha256:e1fd250ce83d94603e9887ec991156a6c26905a6b0001039b7a43699018c0733 as final

ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/London"

COPY --from=build --chown=65532:65532 /opt/dogecoin/depends/x86_64-pc-linux-gnu/bin/ /opt/dogecoin/bin/
COPY --from=build --chown=65532:65532 /opt/dogecoin/.dogecoin/ /opt/dogecoin/.dogecoin/

ENTRYPOINT ["/opt/dogecoin/bin/dogecoind", "-conf=/opt/dogecoin/.dogecoin/dogecoin.conf", "-datadir=/opt/dogecoin/.dogecoin/db", "-printtoconsole", "-txindex=1"]
