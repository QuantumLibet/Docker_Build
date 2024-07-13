## iperf3 v3.17.1, dockerized
### for Raspberry Pi's (RPi1 & RPi2: `arm32v6`/`armel`, RPi2 & RPi3: `arm32v7`/`armhf`, RPi4 & RPi5: `arm64`/`aarch64`) and `x86_64` (Intel/AMD-based) Linux / macOS / Windows WSL

<br/>

### Run as server in the background with:

    docker run --detach --restart unless-stopped --name="iperf3" --publish="5201:5201/tcp" --publish="5201:5201/udp" bridgehead/iperf3

or more concise:

    docker run -d -p 5201:5201 bridgehead/iperf3

<br>

### or use it like any other locally installed executable, by adding iperf3-specific switches after the image name, like:

**a server** running in the foreground with additional switches:

    docker run -p 5201:5201/tcp -p 5201:5201/udp bridgehead/iperf3 --server --one-off

**a client** that contacts server with the ip 192.168.0.1:

    docker run --rm bridgehead/iperf3 -c 192.168.0.1

or **get help** with:

    docker run --rm bridgehead/iperf3 --help

</br>

### Notes:

This version of `iperf3` should run on basically any Raspberry Pi or x86_64-based Linux (Intel/AMD) distro.</br>
With 2,5 MB compressed size, it's smol.</br>
Uncompressed, it takes around 5 MB, only including the alpine base image and the one binary 'iperf3'. Still smol.</br></br>
It uses 600 KB of RAM when running as a server. Tiny.</br></br>
Without any parameters, the container starts `iperf3` in _ server/daemon mode _, waiting for connections from clients.</br>
Nothing else to configure. It just works.

All credits go to Esnet, see: <https://iperf.fr>!

</br>
</br>

### Dockerfile
```xml
# syntax=docker/dockerfile:1

#  Source:  https://github.com/esnet/iperf

FROM --platform=linux/arm/v6  alpine:3.12 AS alpine_armv6
FROM --platform=linux/arm/v7       alpine AS alpine_armv7
FROM --platform=${TARGETPLATFORM}  alpine AS alpine_arm64
FROM --platform=${TARGETPLATFORM}  alpine AS alpine_amd64
FROM alpine_${TARGETARCH}${TARGETVARIANT} AS base

ARG VERSION

LABEL org.opencontainers.image.title="iPerf"
LABEL org.opencontainers.image.description="iperf3 based on alpine"
LABEL org.opencontainers.image.vendor="https://iperf.fr"
LABEL org.opencontainers.image.version="v${VERSION}"
LABEL org.opencontainers.image.revision="https://github.com/esnet/iperf/releases/tag/3.17.1"
LABEL org.opencontainers.image.license="BSD-3-Clause-Clear"
LABEL org.opencontainers.image.authors="container@bridgehead.it"
LABEL org.opencontainers.image.url="https://hub.docker.com/repository/docker/bridgehead/iperf3"
LABEL org.opencontainers.image.source="https://hub.docker.com/repository/docker/bridgehead/iperf3"
LABEL org.opencontainers.image.documentation="https://hub.docker.com/repository/docker/bridgehead/iperf3"
LABEL org.opencontainers.image.base.digest=""
LABEL org.opencontainers.image.base.name=""
LABEL org.opencontainers.image.ref.name=""

EXPOSE 5201

RUN apk --no-cache add iperf3

ENTRYPOINT ["/usr/bin/iperf3"]
CMD ["--server"]
```

Note that the RPi1/2, aka `arm32v6` aka `armel` image explicitly requires the alpine release v3.12. Versions after that have different requirements (esp. time64). See here for more: https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.13.0</br>
Alpine v3.12 is essentially the last version to be usable on old Raspberry Pi 1's and 2's.

</br>
</br>

### Build

```
docker buildx build --builder=container --push --no-cache --build-arg VERSION=3.17.1 --tag bridgehead/iperf3:latest --platform linux/arm/v6,linux/arm/v7,linux/arm64,linux/amd64 --file path/to/Dockerfile .
```

<br/>

### Compose

```
name: iperf3
services:
  iperf3:
    image: docker.io/bridgehead/iperf3:latest
    container_name: iperf3
    ports:
      - "5201:5201"
    restart: unless-stopped
```

Have fun, and a very nice day!
