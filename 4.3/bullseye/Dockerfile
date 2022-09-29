FROM debian:11 AS builder

RUN sed -e '/bullseye main/d' -i /etc/apt/sources.list && \
  echo "deb http://deb.debian.org/debian bullseye main non-free" >> /etc/apt/sources.list

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
    libvpx-dev \
    libwebp-dev \
    libx264-dev \
    libx265-dev \
    pkg-config \
    wget \
    yasm \
    zlib1g-dev

WORKDIR /usr/local/src
RUN git clone https://salsa.debian.org/multimedia-team/zimg.git
WORKDIR /usr/local/src/zimg
RUN git checkout -b 3.0.4 debian/3.0.4+ds1-1
RUN sed -e 's/debhelper-compat (= 13)/debhelper-compat (= 12)/' -i debian/control
RUN gbp buildpackage --git-ignore-new --git-ignore-branch --git-no-pristine-tar --git-builder="debuild -i -I -uc -us"
WORKDIR /usr/local/src
RUN apt-get install ./libzimg2_*_amd64.deb ./libzimg-dev_*_amd64.deb

ARG FFMPEG_VERSION=4.3.4
ARG FFMPEG_SHA256SUM=ba7671eac3e2e0d2c62ea3db820eba07dfba2e9084d37171895efca20b247474

WORKDIR /usr/local/src
RUN wget -q https://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.bz2 && \
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
    --enable-libwebp \
    --enable-libx264 \
    --enable-libx265 \
    --enable-libzimg \
    --enable-nonfree \
    --enable-openssl
RUN make

RUN mkdir -p /tmp/ffmpeg.deb.build/DEBIAN
RUN make install DESTDIR=/tmp/ffmpeg.deb.build -j

COPY 4.3/bullseye/deb-control /tmp/ffmpeg.deb.build/DEBIAN/control
RUN sed -i -e "s/@VERSION@/${FFMPEG_VERSION}/ ; s/@BUILD@/$(date +%Y%m%d%H%M%S)/" /tmp/ffmpeg.deb.build/DEBIAN/control
RUN dpkg -b /tmp/ffmpeg.deb.build /tmp/ffmpeg-${FFMPEG_VERSION}.deb

FROM debian:11-slim

RUN sed -e '/bullseye main/d' -i /etc/apt/sources.list && \
  echo "deb http://deb.debian.org/debian bullseye main non-free" >> /etc/apt/sources.list

COPY --from=builder /tmp/ffmpeg-*.deb /usr/local/src/libzimg*.deb /opt/ffmpeg/
RUN apt-get update && \
  apt-get install -y /opt/ffmpeg/ffmpeg-*.deb /opt/ffmpeg/libzimg2_*_amd64.deb && \
  rm -rf /var/lib/apt/lists/*

ENV PATH=${PATH}:/opt/ffmpeg/bin
