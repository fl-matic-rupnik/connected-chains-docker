FROM debian:trixie-slim@sha256:a99cfc517144bc59b1978475ec53b46ecabec7e43635402ee5b77cc54cd1b20a AS build

ARG VERSION=3.4.1
ARG DEB_REVISION=1
ARG XRPLD_DEB_SHA256=cae8ce3b9bc9451b19975c890714ba789d2004987cdfff9cbd55522c612c6f26

ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/London"

RUN apt-get -y update && apt-get -y install --no-install-recommends \
        ca-certificates \
        curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/xrpld

# https://packages.xrplf.org - DEB/RPM packages are signed with an XRPLF key,
# see https://xrpl.org/docs/infrastructure/installation/install-rippled-on-ubuntu
RUN curl -fsSL -o xrpld.deb "https://packages.xrplf.org/repository/deb-stable/pool/x/xrpld/xrpld_${VERSION}-${DEB_REVISION}_amd64.deb" \
    && echo "${XRPLD_DEB_SHA256}  xrpld.deb" | sha256sum -c - \
    && dpkg-deb -x xrpld.deb /tmp/xrpld/extracted

RUN mkdir -p /opt/xrpld/bin /opt/xrpld/.xrpld/db \
    && cp /tmp/xrpld/extracted/usr/bin/xrpld /tmp/xrpld/extracted/usr/bin/validator-keys /opt/xrpld/bin/

COPY xrpld.cfg validators.txt /opt/xrpld/.xrpld/

FROM gcr.io/distroless/base-debian13:nonroot@sha256:ab7554b6d07ad354fad31957f8a1a813e65dfb93a8ad160568c79c3f2be6884f AS final

ENV DEBIAN_FRONTEND="noninteractive" TZ="Europe/London"

COPY --from=build --chown=65532:65532 /opt/xrpld/bin/ /opt/xrpld/bin/
COPY --from=build --chown=65532:65532 /opt/xrpld/.xrpld/ /opt/xrpld/.xrpld/

# https://xrpl.org/commandline-usage.html
ENTRYPOINT ["/opt/xrpld/bin/xrpld", "--conf=/opt/xrpld/.xrpld/xrpld.cfg", "--fg"]
