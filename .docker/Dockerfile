FROM alpine:latest AS builder

LABEL maintainer="AUTUMN"

ARG NGINX_VERSION=1.16.1
ARG RTMP_MODULE_VERSION=1.2.1

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

RUN apk update && \
  apk add \
  git  \
  gcc  \
  binutils  \
  gmp  \
  isl  \
  libgomp  \
  libatomic  \
  libgcc  \
  openssl  \
  pkgconf  \
  pkgconfig  \
  mpfr3  \
  mpc1  \
  libstdc++  \
  ca-certificates  \
  libssh2  \
  curl  \
  expat  \
  pcre  \
  musl-dev  \
  libc-dev  \
  pcre-dev  \
  zlib-dev  \
  openssl-dev  \
  curl  \
  make

WORKDIR /tmp

RUN wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
  wget https://github.com/arut/nginx-rtmp-module/archive/v${RTMP_MODULE_VERSION}.tar.gz && \
  tar -xf nginx-${NGINX_VERSION}.tar.gz && \
  tar -xf v${RTMP_MODULE_VERSION}.tar.gz && \ 
  mv nginx-rtmp-module-${RTMP_MODULE_VERSION} nginx-rtmp-module

RUN cd nginx-${NGINX_VERSION} && \
  ./configure \
  --prefix=/etc/nginx \
  --with-http_ssl_module  \
  --add-module=../nginx-rtmp-module \
  --with-cc-opt="-Wimplicit-fallthrough=0" && \
  make && \
  make install


FROM alpine:latest

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

RUN set -x \
# create nginx user/group first, to be consistent throughout docker variants
    && addgroup -g 101 -S nginx \
    && adduser -S -D -H -u 101 -h /var/cache/nginx -s /sbin/nologin -G nginx -g nginx nginx

RUN  apk --no-cache add \
  openssl \
  libstdc++ \
  ca-certificates \
  pcre \
  tzdata && \
  rm -rf /var/cache/apk/*

# set the localtime
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \ 
  echo "Asia/Shanghai" > /etc/timezone && \
  mkdir -p /etc/nginx/logs && \
  # touch /etc/nginx/logs/access.log && touch /etc/nginx/logs/error.log && \
  mkdir -p /app/wwwroot /app/stream && \
  # forward request and error logs to docker log collector
  ln -sf /dev/stdout /etc/nginx/logs/access.log && \
  ln -sf /dev/stderr /etc/nginx/logs/error.log

COPY --from=builder /etc/nginx /etc/nginx
COPY --from=builder /tmp/nginx-rtmp-module/stat.xsl /app/stream/stat.xsl

EXPOSE 1935
EXPOSE 80

CMD ["/etc/nginx/sbin/nginx", "-g", "daemon off;"]