FROM debian:10 AS builder

RUN sed -e '/buster main/d' -i /etc/apt/sources.list && \
  echo "deb http://deb.debian.org/debian buster main non-free" >> /etc/apt/sources.list

RUN apt-get update && \
  apt-get install -y \
    autoconf \
    automake \
    build-essential \
    ca-certificates \
    libaom-dev \
    libfdk-aac-dev \
    libmp3lame-dev \
    libnuma-dev \
    libopus-dev \
    libssl-dev \
    libtheora-dev \
    libtool \
    libvorbis-dev \
    libvpx-dev \
    libx264-dev \
    libx265-dev \
    pkg-config \
    wget \
    yasm \
    zlib1g-dev

ARG FFMPEG_VERSION=4.2.2
ARG FFMPEG_SHA256SUM=b620d187c26f76ca19e74210a0336c3b8380b97730df5cdf45f3e69e89000e5c

WORKDIR /usr/local/src
RUN wget -q http://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
  echo "${FFMPEG_SHA256SUM}  ffmpeg-${FFMPEG_VERSION}.tar.bz2" | sha256sum -c && \
  tar xf ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
  rm ffmpeg-${FFMPEG_VERSION}.tar.bz2
COPY ffmpeg-patches ./ffmpeg-patches

WORKDIR /usr/local/src/ffmpeg-${FFMPEG_VERSION}
RUN cat ../ffmpeg-patches/* | patch -p1
RUN ./configure \
    --prefix=/opt/ffmpeg \
    --disable-doc \
    --disable-ffplay \
    --disable-shared \
    --enable-gpl \
    --enable-libaom \
    --enable-libfdk_aac \
    --enable-libmp3lame \
    --enable-libopus \
    --enable-libvorbis \
    --enable-libvpx \
    --enable-libx264 \
    --enable-libx265 \
    --enable-nonfree \
    --enable-openssl
RUN make

RUN mkdir -p /tmp/ffmpeg.deb.build/DEBIAN
RUN make install DESTDIR=/tmp/ffmpeg.deb.build

COPY 4.2/buster/deb-control /tmp/ffmpeg.deb.build/DEBIAN/control
RUN sed -i -e "s/@VERSION@/${FFMPEG_VERSION}/ ; s/@BUILD@/$(date +%Y%m%d%H%M%S)/" /tmp/ffmpeg.deb.build/DEBIAN/control
RUN dpkg -b /tmp/ffmpeg.deb.build /tmp/ffmpeg-${FFMPEG_VERSION}.deb

FROM debian:10-slim

RUN echo "deb http://deb.debian.org/debian buster main non-free" >> /etc/apt/sources.list

COPY --from=builder /tmp/ffmpeg-*.deb /opt/ffmpeg/
RUN apt-get update && \
  apt-get install -y /opt/ffmpeg/ffmpeg-*.deb && \
  rm -rf /var/lib/apt/lists/*

ENV PATH=${PATH}:/opt/ffmpeg/bin
