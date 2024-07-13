# syntax=docker/dockerfile:1

#  Source:  https://security.appspot.com/vsftpd.html
#  Source:  https://github.com/onjin/docker-alpine-vsftpd

FROM --platform=linux/arm/v6  alpine:3.12 AS alpine_armv6
FROM --platform=linux/arm/v7       alpine AS alpine_armv7
FROM --platform=${TARGETPLATFORM}  alpine AS alpine_arm64
FROM --platform=${TARGETPLATFORM}  alpine AS alpine_amd64
FROM alpine_${TARGETARCH}${TARGETVARIANT} AS base

ARG VERSION

LABEL org.opencontainers.image.title="vsftpd"
LABEL org.opencontainers.image.description="vsftpd based on alpine"
LABEL org.opencontainers.image.vendor="https://security.appspot.com/vsftpd.html"
LABEL org.opencontainers.image.version="v${VERSION}"
LABEL org.opencontainers.image.revision=""
LABEL org.opencontainers.image.license="GPL-1.0-or-later"
LABEL org.opencontainers.image.authors="container@bridgehead.it"
LABEL org.opencontainers.image.url="https://hub.docker.com/repository/docker/bridgehead/vsftpd"
LABEL org.opencontainers.image.source="https://hub.docker.com/repository/docker/bridgehead/vsftpd"
LABEL org.opencontainers.image.documentation="https://hub.docker.com/repository/docker/bridgehead/vsftpd"
LABEL org.opencontainers.image.base.digest=""
LABEL org.opencontainers.image.base.name=""
LABEL org.opencontainers.image.ref.name=""

#EXPOSE 20/tcp
EXPOSE 21/tcp
EXPOSE 10091-10100

#  expose /etc/vsftpd for additional configuration during runtime
VOLUME /etc/vsftpd
#  expose /home/vsftpd for uploads
VOLUME /home/vsftpd

RUN adduser -s /bin/false -D vsftpd
RUN echo "vsftpd:vsftpd" | chpasswd

RUN apk --no-cache add vsftpd tzdata
RUN printf "%s\n"                \
    'anonymous_enable=no'        \
    'local_enable=yes'           \
    'write_enable=yes'           \
    'chmod_enable=no'            \
    'local_umask=022'            \
    'use_localtime=yes'          \
    'idle_session_timeout=300'   \
    'nopriv_user=vsftpd'         \
    'chroot_local_user=yes'      \
    'passwd_chroot_enable=yes'   \
    'allow_writeable_chroot=yes' \
    'seccomp_sandbox=no'         \
    'max_clients=4'              \
    'max_per_ip=2'               \
    'pasv_max_port=10100'        \
    'pasv_min_port=10091'        \
    > /etc/vsftpd/vsftpd.conf

CMD ["vsftpd", "/etc/vsftpd/vsftpd.conf"]


#  docker buildx build --builder=container --push --no-cache --build-arg VERSION=3.0.5-r2 --tag bridgehead/vsftpd:latest --platform linux/arm/v6,linux/arm/v7,linux/arm64,linux/amd64 --file path/to/Dockerfile .


#  default passwd is 'vsftpd'
#  change password with:  docker exec vsftpd sh -c 'echo vsftpd:NEWPASSWORD | chpasswd'
#  if you need a random password, use:  LC_ALL=C tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 8


#  test logins with:  ftp 127.0.0.1
#  remember to manually switch to PASV mode with `pass` if you happen to use an older ftp client
