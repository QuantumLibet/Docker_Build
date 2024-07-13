###  >>> IF YOU DO NOT KNOW WHAT THIS IS, YOU DON'T NEED IT! <<<
Really. Don't worry.

</br>

## vlmcsd v1113-5, dockerized
### for Raspberry Pi's (RPi1 & RPi2: `arm32v6`/`armel`, RPi2 & RPi3: `arm32v7`/`armhf`, RPi4 & RPi5: `arm64`/`aarch64`) and `x86_64` (Intel/AMD-based) Linux / macOS / Windows WSL

</br>

### Run as server in the background with:

    docker run --detach --rm --name="vlmcsd" --publish="1688:1688/tcp" bridgehead/vlmcsd

or more concise:

    docker run -d -p 1688:1688 bridgehead/vlmcsd

### or use it like any other locally installed executable, by adding `vlmcsd`-specific switches after the image name, like:

**a server** running in the foreground with additional switches:

    docker run -p 1688:1688 bridgehead/vlmcsd /vlmcsd -D -d -e -v

**a client** that contacts the above server at some ip:

    docker run --rm bridgehead/vlmcsd /vlmcs 192.168.0.1

or **get help** with:

    docker run --rm bridgehead/vlmcsd /vlmcsd --help

or

    docker run --rm bridgehead/vlmcsd /vlmcs --help

</br>

### Notes:

This version of `vlmcs`/`vlmcsd` should run on basically any Raspberry Pi or x86_64-based Linux (Intel/AMD) distro.</br>
With 2,5 MB compressed size (depending on the CPU architecture), it's smol.</br>
Uncompressed, it takes around 5-8 MB, only including the alpine base image and the two binaries `vlmcs` & `vlmcsd`. Still smol.</br>
It uses 256 KB of RAM when running as a server. Tiny. </br></br>
Without any parameters, the container starts `vlmcsd` in _ server mode _, waiting for connections from clients.</br>
Nothing to configure. It just works.

### All credits and my deepest gratitude go to Hotbird64!

</br>
</br>

### Dockerfile
```xml
# syntax=docker/dockerfile:1

#  Source:  https://github.com/Wind4/vlmcsd.git

FROM --platform=linux/arm/v6  alpine:3.12 AS alpine_armv6
FROM --platform=linux/arm/v7       alpine AS alpine_armv7
FROM --platform=${TARGETPLATFORM}  alpine AS alpine_arm64
FROM --platform=${TARGETPLATFORM}  alpine AS alpine_amd64
FROM alpine_${TARGETARCH}${TARGETVARIANT} AS build

WORKDIR /root

RUN apk --no-cache add build-base gcc abuild binutils cmake git
RUN git clone https://github.com/Wind4/vlmcsd.git .
RUN make
RUN chmod +x bin/vlmcs*

#========================================================

FROM alpine_${TARGETARCH}${TARGETVARIANT} AS base

ARG VERSION

LABEL org.opencontainers.image.title="vlmcsd"
LABEL org.opencontainers.image.description="vlmcsd based on alpine"
LABEL org.opencontainers.image.vendor="https://github.com/Wind4/vlmcsd.git"
LABEL org.opencontainers.image.version="v${VERSION}"
LABEL org.opencontainers.image.revision="https://github.com/Wind4/vlmcsd/releases"
LABEL org.opencontainers.image.license=""
LABEL org.opencontainers.image.authors="container@bridgehead.it"
LABEL org.opencontainers.image.url="https://hub.docker.com/repository/docker/bridgehead/vlmcsd"
LABEL org.opencontainers.image.source="https://hub.docker.com/repository/docker/bridgehead/vlmcsd"
LABEL org.opencontainers.image.documentation="https://hub.docker.com/repository/docker/bridgehead/vlmcsd"
LABEL org.opencontainers.image.base.digest=""
LABEL org.opencontainers.image.base.name=""
LABEL org.opencontainers.image.ref.name=""

EXPOSE 1688

COPY --from=build /root/bin/vlmcs* /

#  use as server (default)
#  -D  daemonize-not, keep in foreground
#  -d  disconnect clients after each request
#  -e  log to stdout
#  -v  log verbose
CMD ["/vlmcsd","-D","-d","-e","-v"]

#  to use as client: docker run --rm bridgehead/vlmcsd /vlmcs <ip_of_vlmcsd-server>
```

Note that the RPi1/2, aka `arm32v6` aka `armel` image explicitly requires the alpine release v3.12. Versions after that have different requirements (esp. time64). See here for more: https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.13.0</br>
Alpine v3.12 is essentially the last version to be usable on old Raspberry Pi 1's and 2's.

</br>
</br>

### Build

```
docker buildx build --builder=container --push --no-cache --build-arg VERSION=1113 --tag bridgehead/vlmcsd:latest --platform linux/arm/v6,linux/arm/v7,linux/arm64,linux/amd64 --file path/to/Dockerfile .
```

</br>

### Compose

```
name: vlmcsd
services:
  vlmcsd:
    image: docker.io/bridgehead/vlmcsd:latest
    container_name: vlmcsd
    ports:
      - "1688:1688/tcp"
    restart: unless-stopped
```

Have fun and a very nice day!
