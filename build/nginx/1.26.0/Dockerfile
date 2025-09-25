# ---------- 构建阶段 ----------
FROM alpine:3.18 AS builder

ARG NGINX_VERSION=1.26.0

# 安装构建依赖
RUN apk add --no-cache \
      gcc \
      g++ \
      make \
      openssl \
      openssl-dev \
      pcre \
      pcre-dev \
      zlib \
      zlib-dev \
      libxslt-dev \
      linux-headers \
      curl \
      tar

# 下载 & 编译 nginx
RUN curl -o /tmp/nginx-${NGINX_VERSION}.tar.gz http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
   && tar -zxvf /tmp/nginx-${NGINX_VERSION}.tar.gz -C /tmp/ \
   && cd /tmp/nginx-${NGINX_VERSION} \
   && ./configure --prefix=/usr/local/nginx \
      --user=nginx \
      --group=nginx \
      --with-http_stub_status_module \
      --with-http_ssl_module \
      --with-http_v2_module \
      --with-http_realip_module \
      --with-http_addition_module \
      --with-http_sub_module \
      --with-http_dav_module \
      --with-http_flv_module \
      --with-http_mp4_module \
      --with-http_gunzip_module \
      --with-http_gzip_static_module \
      --with-http_random_index_module \
      --with-http_secure_link_module \
      --with-http_auth_request_module \
      --with-http_xslt_module \
      --with-mail \
      --with-mail_ssl_module \
      --with-file-aio \
   && make && make install

# ---------- 运行阶段 ----------
FROM alpine:3.18

ARG NGINX_VERSION=1.26.0
ENV PATH="${PATH}:/usr/local/nginx/sbin"

# 创建 nginx 用户和时区设置
RUN addgroup -S nginx \
   && adduser -S -G nginx -s /sbin/nologin nginx \
   && apk add --no-cache tzdata openssl pcre zlib libxslt curl \
   && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
   && echo 'Asia/Shanghai' > /etc/timezone \
   && rm -rf /var/cache/apk/*

# 从 builder 阶段拷贝 nginx
COPY --from=builder /usr/local/nginx /usr/local/nginx
RUN ln -s /usr/local/nginx/sbin/* /usr/local/sbin

WORKDIR /usr/local/nginx

EXPOSE 80 443

# 默认启动 nginx 前台运行，用户可覆盖 CMD 参数
CMD ["nginx", "-g", "daemon off;"]
