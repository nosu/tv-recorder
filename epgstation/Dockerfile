# FFmpeg
FROM alpine:3.11 AS ffmpeg-build

ENV FFMPEG_VERSION=4.2.2
ENV PREFIX=/usr/local/ffmpeg
ENV LD_LIBRARY_PATH=/usr/lib

RUN set -eux
RUN apk add --no-cache \
	autoconf \
	automake \
	build-base \
	ca-certificates \
	coreutils \
	curl \
	freetype-dev \
	gcc \
	lame-dev \
	libtool \
	libogg-dev \
	libass \
	libass-dev \
	libvpx-dev \
	libvorbis-dev \
	libwebp-dev \
	libtheora-dev \
	openssl-dev \
	opus-dev \
	pkgconf \
	rtmpdump-dev \
	x264-dev \
	x265-dev \
	yasm

RUN echo http://dl-cdn.alpinelinux.org/alpine/edge/testing >> /etc/apk/repositories && \
	apk add --no-cache fdk-aac-dev

RUN mkdir -p /build/aribb24
WORKDIR /build/aribb24
RUN curl -fsSL https://github.com/nkoriyama/aribb24/tarball/master \
	| tar -xz --strip-components=1
RUN autoreconf -fiv
RUN ./configure --prefix=/usr/local/ffmpeg --enable-static --disable-shared
RUN make
RUN make install

RUN mkdir -p /build/ffmpeg
WORKDIR /build/ffmpeg
RUN curl -fsSL http://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.gz \
	| tar -xz --strip-components=1

RUN PKG_CONFIG_PATH=/usr/local/ffmpeg/lib/pkgconfig ./configure \
	--enable-version3 \
	--enable-gpl \
	--enable-nonfree \
	--enable-small \
	--enable-libmp3lame \
	--enable-libx264 \
	--enable-libx265 \
	--enable-libvpx \
	--enable-libtheora \
	--enable-libvorbis \
	--enable-libopus \
	--enable-libfdk-aac \
	--enable-libass \
	--enable-libwebp \
	--enable-librtmp \
	--enable-libaribb24 \
	--enable-postproc \
	--enable-avresample \
	--enable-libfreetype \
	--enable-openssl \
	--disable-debug \
	--disable-doc \
	--disable-ffplay \
	--extra-cflags="-I${PREFIX}/include" \
	--extra-ldflags="-L${PREFIX}/lib" \
	--extra-libs="-lpthread -lm" \
	--prefix="${PREFIX}"
RUN make -j $(nproc)
RUN make install
RUN make distclean

RUN rm -rf /tmp/* /var/cache/apk/*


# EPGStation
FROM node:12-alpine
LABEL maintainer "collelog <collelog.cavamin@gmail.com>"

EXPOSE 8888
EXPOSE 8889

WORKDIR /usr/local/

RUN set -eux && \
	apk upgrade --update && \
	apk add --no-cache ca-certificates pcre lame libogg libass libvpx libvorbis libwebp libtheora openssl opus rtmpdump x264-dev x265-dev && \
	apk add --no-cache --virtual build-packages ca-certificates curl g++ make python3 tzdata && \
	\
	# timezone
	cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
	echo "Asia/Tokyo" > /etc/timezone && \
	\
	# install EPGStation
	mkdir -p /usr/local/EPGStation && \
	cd /usr/local/EPGStation && \
	curl -fsSL https://github.com/l3tnun/EPGStation/tarball/master \
		| tar -xz --strip-components=1 && \
	cd /usr/local/EPGStation && \
	npm install && \
	npm run build && \
	\
	# cleaning
	npm cache verify && \
	apk del --purge build-packages && \
	rm -rf /tmp/* ~/.npm /var/cache/apk/*

COPY --from=ffmpeg-build /usr/local/ffmpeg /usr/local/ffmpeg
COPY --from=ffmpeg-build /usr/lib/libfdk-aac.so.2 /usr/lib/libfdk-aac.so.2

WORKDIR /usr/local/EPGStation

ENTRYPOINT npm start
