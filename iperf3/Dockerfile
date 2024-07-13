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


# docker buildx build --push --no-cache --build-arg VERSION=3.17.1 --tag bridgehead/iperf3:arm32v6 --platform linux/arm/v6 --file path/to/Dockerfile .
# docker buildx build --push --no-cache --build-arg VERSION=3.17.1 --tag bridgehead/iperf3:arm32v7 --platform linux/arm/v7 --file path/to/Dockerfile .
# docker buildx build --push --no-cache --build-arg VERSION=3.17.1 --tag bridgehead/iperf3:arm64   --platform linux/arm64  --file path/to/Dockerfile .
# docker buildx build --push --no-cache --build-arg VERSION=3.17.1 --tag bridgehead/iperf3:amd64   --platform linux/amd64  --file path/to/Dockerfile .
#
# docker manifest create bridgehead/iperf3:latest --amend bridgehead/iperf3:arm32v6 --amend bridgehead/iperf3:arm32v7 --amend bridgehead/iperf3:arm64 --amend bridgehead/iperf3:amd64
# docker manifest push bridgehead/iperf3:latest
#
#or
#
# docker buildx build --builder=container --push --no-cache --build-arg VERSION=3.17.1 --tag bridgehead/iperf3:buildx-latest --platform linux/arm/v6,linux/arm/v7,linux/arm64,linux/amd64 --file path/to/Dockerfile .

# to get help:                               docker run --rm bridgehead/iperf3 --help
# to use as client:                          docker run --rm bridgehead/iperf3 --client --time 120 --no-delay
# to use as server with additional switches: docker run --rm bridgehead/iperf3 --server --format M --one-off
