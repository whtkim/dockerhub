FROM alpine:3.18

ARG NGINX_VERSION=1.26.0

ENV PATH="${PATH}:/usr/local/nginx/sbin"

# 系统设置
RUN addgroup -S nginx \
 && adduser -S -G nginx -s /sbin/nologin nginx \
 && apk add --no-cache tzdata \
 && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
 && echo 'Asia/Shanghai' > /etc/timezone

# 安装依赖和 nginx
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
    linux-headers \
    curl \
 && curl -o /tmp/nginx-${NGINX_VERSION}.tar.gz http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz \
 && tar -zxvf /tmp/nginx-${NGINX_VERSION}.tar.gz -C /tmp/ \
 && cd /tmp/nginx-${NGINX_VERSION} \
 && ./configure --prefix=/usr/local/nginx \
    --user=nginx \
    --group=nginx \
    --with-http_stub_status_module \
    --with-http_ssl_module \
 && make && make install \
 && ln -s /usr/local/nginx/sbin/* /usr/local/sbin \
 && rm -rf /tmp/nginx-* \
 && apk del gcc g++ make curl \
 && rm -rf /var/cache/apk/*

# 设置工作目录
WORKDIR /usr/local/nginx

# 暴露端口
EXPOSE 80

# 执行 nginx
ENTRYPOINT ["nginx", "-c", "/usr/local/nginx/conf/nginx.conf", "-g", "daemon off;"]