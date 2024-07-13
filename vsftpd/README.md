## vsftpd 3.0.5-r2, dockerized
### for Raspberry Pi's (RPi1 & RPi2: `arm32v6`/`armel`, RPi2 & RPi3: `arm32v7`/`armhf`, RPi4 & RPi5: `arm64`/`aarch64`) and `x86_64` (Intel/AMD-based) Linux / macOS / Windows WSL

</br>

### Run as server in the background with:

    docker run --detach --restart unless-stopped --name="vsftpd" --publish="21:21/tcp" --publish="10091-10100:10091-10100/tcp" --volume "local/path/to/vsftpd/data:/home/vsftpd" bridgehead/vsftpd

or more concise:

    docker run -d -p 21:21 -p 10090-10100:10090-10100 -v "local/path/to/vsftpd/data:/home/vsftpd" bridgehead/vsftpd

</br>

### Notes:

`local/path/to/vsftpd/data` is a path in your local file system where the data that comes in is stored (or should be read from). Could be as simple as: `~/ftp-data`

`local/path/to/vsftpd/config` [optional!] is a path in your local file system where you would like to put a vsftpd.conf file to configure vsftpd. Eg. `~/ftp-conf`

User and password are both: 'vsftpd'.</br>
Change the password with: `docker exec vsftpd sh -c 'echo vsftpd:NEWPASSWORD | chpasswd'`
For an inspiration, use: `LC_ALL=C tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 8`

Test logins with: `ftp 127.0.0.1`
Remember to manually switch to PASV mode with `pass` after login, if you happen to use an older ftp client.

This version of `vsftpd` should run on basically any Raspberry Pi or x86_64-based Linux (Intel/AMD) distro.</br>
With around 3.5 MB compressed size, it's not big. </br>
Uncompressed, it takes around 6 MB, only including the alpine base image and the one binary 'vsftpd'. Still smol. But double that on arm64. :(</br></br>
It uses 700 KB of RAM when running as a server. Tiny.</br></br>

All credits go to [Chris Evans](https://security.appspot.com/vsftpd.html) and [Marek Wyvial](https://github.com/onjin), where you'll find more information.

</br>
</br>

### Dockerfile
```xml
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

#EXPOSE 20
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
```

</br>
</br>

Note that the RPi1/2, aka `arm32v6` aka `armel` image explicitly requires the alpine release v3.12. Versions after that have different requirements (esp. time64). See here for more: https://wiki.alpinelinux.org/wiki/Release_Notes_for_Alpine_3.13.0</br>
Alpine v3.12 is essentially the last version to be usable on old Raspberry Pi 1's and 2's.

</br>
</br>

### Build

```
docker buildx build --builder=container --push --no-cache --build-arg VERSION=3.0.5-r2 --tag bridgehead/vsftpd:latest --platform linux/arm/v6,linux/arm/v7,linux/arm64,linux/amd64 --file path/to/Dockerfile .
```

</br>

### Compose

```
name: vsftpd
services:
    vsftpd:
      image: docker.io/bridgehead/vsftpd:latest
      container_name: vsftpd
      ports:
        - "21:21"
        - "10091-10100:10091-10100"
      environment:
        - TZ=Europe/Berlin
      volumes:
        #- ~/.config/vsftpd:/etc/vsftpd
        - ~/.config/vsftpd:/home/vsftpd
      restart: unless-stopped
```
Have fun, and a very nice day!
