# recpt1
FROM alpine:3.11 AS recpt1-build

RUN set -eux
RUN apk upgrade --update
RUN apk add --no-cache ca-certificates curl git gcc g++ make autoconf automake pcsc-lite-dev

RUN mkdir -p /build
WORKDIR /build
RUN curl -fsSL https://github.com/stz2012/recpt1/tarball/master \
	| tar -xz --strip-components=1

WORKDIR /build/recpt1
RUN ./autogen.sh
RUN ./configure --prefix=/usr/local
RUN make -j $(nproc)
RUN make install


# arib-b25-stream-test
FROM node:12-alpine3.11 AS arib-b25-stream-test-build

RUN apk upgrade --update
RUN apk add --no-cache gcc g++ make libc-dev pcsc-lite-dev pkgconfig

RUN npm install arib-b25-stream-test -g --unsafe


# final image
FROM node:12-alpine3.11
LABEL maintainer "collelog <collelog.cavamin@gmail.com>"

EXPOSE 40772
ENV DOCKER="YES"

# recpt1
COPY --from=recpt1-build /usr/local/bin/recpt1 /usr/local/bin/recpt1

# arib-b25-stream-test
COPY --from=arib-b25-stream-test-build /usr/local/lib/node_modules/arib-b25-stream-test /usr/local/lib/node_modules/arib-b25-stream-test

RUN set -eux && \
	apk upgrade --update && \
	apk add --no-cache ccid libstdc++ pcsc-lite pcsc-lite-libs tzdata && \
	\
	# timezone
	cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && \
	echo "Asia/Tokyo" > /etc/timezone && \
	apk del tzdata && \
	\
	# arib-b25-stream-test
	ln -sf /usr/local/lib/node_modules/arib-b25-stream-test/bin/b25 /usr/local/bin/arib-b25-stream-test && \
	\
	# pm2
	npm install pm2 -g && \
	\
	# mirakurun
	npm install mirakurun@latest -g --unsafe --production && \
	sed -i -e 's/--max_old_space_size=512/--max_old_space_size=1024/g' /usr/local/lib/node_modules/mirakurun/package.json && \
	\
	# cleaning
	npm cache verify && \
	rm -rf /tmp/* ~/.npm /var/cache/apk/*

COPY services.sh /usr/local/bin

WORKDIR /usr/local/lib/node_modules/mirakurun

ENTRYPOINT /usr/local/bin/services.sh
