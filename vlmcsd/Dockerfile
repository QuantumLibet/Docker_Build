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


#  docker buildx build --builder=container --push --no-cache --build-arg VERSION=1113 --tag bridgehead/vlmcsd:latest --platform linux/arm/v6,linux/arm/v7,linux/arm64,linux/amd64 --file path/to/Dockerfile .

#  to use as client: docker run --rm bridgehead/vlmcsd /vlmcs <ip_of_vlmcsd-server>
