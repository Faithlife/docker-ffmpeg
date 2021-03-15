FROM debian:10 AS builder

RUN sed -e '/buster main/d' -i /etc/apt/sources.list && \
  echo "deb http://deb.debian.org/debian buster main non-free" >> /etc/apt/sources.list

RUN apt-get update && \
  apt-get install -y \
    autoconf \
    automake \
    build-essential \
    ca-certificates \
    devscripts \
    dh-exec \
    doxygen \
    git-buildpackage \
    libaom-dev \
    libfdk-aac-dev \
    libmp3lame-dev \
    libnuma-dev \
    libopus-dev \
    libssl-dev \
    libtheora-dev \
    libtool \
    libvorbis-dev \
    libx264-dev \
    libx265-dev \
    pkg-config \
    wget \
    yasm \
    zlib1g-dev

WORKDIR /usr/local/src
RUN git clone https://salsa.debian.org/multimedia-team/libvpx.git
WORKDIR /usr/local/src/libvpx
RUN git checkout -b 1.8 debian/1.8.2-1
RUN gbp buildpackage --git-ignore-branch --git-no-pristine-tar --git-builder="debuild -i -I -uc -us"
WORKDIR /usr/local/src
RUN apt-get install ./libvpx6_*_amd64.deb ./libvpx-dev_*_amd64.deb

ARG ZIMG_VERSION=3.0.1
ARG ZIMG_SHA256SUM=c50a0922f4adac4efad77427d13520ed89b8366eef0ef2fa379572951afcc73f

ARG FFMPEG_VERSION=4.2.2
ARG FFMPEG_SHA256SUM=b620d187c26f76ca19e74210a0336c3b8380b97730df5cdf45f3e69e89000e5c

WORKDIR /usr/local/src

RUN wget -q -O libzimg.tar.gz https://github.com/sekrit-twc/zimg/archive/release-${ZIMG_VERSION}.tar.gz && \
  echo "${ZIMG_SHA256SUM} libzimg.tar.gz" | sha256sum -c && \
  tar xf libzimg.tar.gz && \
  rm libzimg.tar.gz

RUN wget -q http://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
  echo "${FFMPEG_SHA256SUM}  ffmpeg-${FFMPEG_VERSION}.tar.bz2" | sha256sum -c && \
  tar xf ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
  rm ffmpeg-${FFMPEG_VERSION}.tar.bz2
COPY ffmpeg-patches ./ffmpeg-patches

WORKDIR /usr/local/src/zimg-release-${ZIMG_VERSION}
RUN ./autogen.sh && \
  ./configure --enable-static --disable-shared && \
  make install

WORKDIR /usr/local/src/ffmpeg-${FFMPEG_VERSION}
RUN cat ../ffmpeg-patches/* | patch -p1
RUN ./configure \
    --pkg-config-flags="--static" \
    --extra-libs="-lm" \
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
    --enable-libzimg \
    --enable-nonfree \
    --enable-openssl
RUN make

RUN mkdir -p /tmp/ffmpeg.deb.build/DEBIAN
RUN make install DESTDIR=/tmp/ffmpeg.deb.build -j

COPY 4.2/buster/deb-control /tmp/ffmpeg.deb.build/DEBIAN/control
RUN sed -i -e "s/@VERSION@/${FFMPEG_VERSION}/ ; s/@BUILD@/$(date +%Y%m%d%H%M%S)/" /tmp/ffmpeg.deb.build/DEBIAN/control
RUN dpkg -b /tmp/ffmpeg.deb.build /tmp/ffmpeg-${FFMPEG_VERSION}.deb

FROM debian:10-slim

RUN echo "deb http://deb.debian.org/debian buster main non-free" >> /etc/apt/sources.list

COPY --from=builder /tmp/ffmpeg-*.deb /usr/local/src/libvpx*.deb /opt/ffmpeg/
RUN apt-get update && \
  apt-get install -y /opt/ffmpeg/ffmpeg-*.deb /opt/ffmpeg/libvpx6_*_amd64.deb && \
  rm -rf /var/lib/apt/lists/*

ENV PATH=${PATH}:/opt/ffmpeg/bin
