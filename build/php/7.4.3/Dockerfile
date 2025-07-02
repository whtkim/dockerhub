# -------------------------------
# 🏗️  1. Builder Stage
# -------------------------------
FROM ubuntu:20.04 AS builder

SHELL ["/bin/bash", "-c"]

ARG php_version="7.4.3"
ARG swoole_version="4.8.12"
ARG imagick_version="3.8.0"
ARG imagemagick_version="7.1.1-47"
ARG redis_version="6.0.2"
ARG swoole_enable=1
ARG imagick_enable=1
ARG timezone="Asia/Shanghai"

ENV PHP_ROOTDIR="/usr/local/php"
ENV DEBIAN_FRONTEND="noninteractive"
ENV PATH="$PATH:${PHP_ROOTDIR}/bin"

# 安装依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    tzdata curl ca-certificates \
    build-essential autoconf pkg-config re2c libtool libxml2-dev \
    libssl-dev libreadline-dev libsqlite3-dev zlib1g-dev libcurl4-openssl-dev \
    libpng-dev libjpeg-dev libwebp-dev libfreetype-dev \
    libsodium-dev libzip-dev libonig-dev libxpm-dev libxslt-dev \
    libmagickwand-dev libmagickcore-dev libbrotli-dev imagemagick \
    && cp /usr/share/zoneinfo/${timezone} /etc/localtime \
    && echo "${timezone}" > /etc/timezone

# 下载并构建 PHP
RUN curl -fsSL -o /tmp/php.tar.gz https://www.php.net/distributions/php-${php_version}.tar.gz \
    && tar -xzf /tmp/php.tar.gz -C /usr/local/src \
    && cd /usr/local/src/php-${php_version} \
    && ./configure --prefix=${PHP_ROOTDIR} \
        --with-config-file-path=${PHP_ROOTDIR} \
        --with-config-file-scan-dir=${PHP_ROOTDIR}/conf.d \
        --with-pdo-mysql=shared,mysqlnd --enable-shared --enable-mysqlnd\
        --enable-gd --with-freetype --with-jpeg --with-webp \
        --with-openssl \
        --with-zlib \
        --with-curl \
        --with-readline \
        --with-mhash \
        --with-sodium \
        --with-xsl \
        --with-zip \
        --with-pear \
        --enable-fpm \
        --enable-mbstring \
        --enable-pcntl \
        --enable-sockets \
        --enable-session \
        --enable-bcmath \
        --enable-intl \
        --enable-soap \
        --enable-opcache \
    && make -j$(nproc) && make install \
    && mkdir -p ${PHP_ROOTDIR}/conf.d \
    && cp -v /usr/local/src/php-${php_version}/php.ini-production ${PHP_ROOTDIR}/php.ini \
    && mv /usr/local/src/php-${php_version}/ext ${PHP_ROOTDIR}/ext

# 安装 redis 扩展 + swoole + imagick
COPY script/php-ext-build ${PHP_ROOTDIR}/bin/php-ext-build
RUN set -e && curl -fsSL -o ${PHP_ROOTDIR}/bin/composer https://mirrors.aliyun.com/composer/composer.phar \
    && chmod +x ${PHP_ROOTDIR}/bin/composer \
    && source ${PHP_ROOTDIR}/bin/php-ext-build \
    && buildRedis "${redis_version}" \
    && [ "$swoole_enable" = "1" ] && buildSwoole "${swoole_version}" \
    && [ "$imagick_enable" = "1" ] && buildImagick "${imagemagick_version}" "${imagick_version}"

# 清理构建产物
RUN apt-get purge -y build-essential autoconf pkg-config re2c libtool \
    && apt-get autoremove -y && rm -rf /var/lib/apt/lists/* /tmp/*
    
# -------------------------------
# 🚀 2. Runtime Stage
# -------------------------------
FROM ubuntu:20.04

ARG timezone="Asia/Shanghai"
ENV DEBIAN_FRONTEND="noninteractive"
ENV PHP_ROOTDIR="/usr/local/php"
ENV PATH="${PATH}:${PHP_ROOTDIR}/bin:${PHP_ROOTDIR}/sbin"

COPY --from=builder /usr/local/php ${PHP_ROOTDIR}
COPY --from=builder /usr/local/imagick /usr/local/imagick
COPY script/php-boot /usr/local/bin/php-boot

RUN echo "/usr/local/imagick/lib" > /etc/ld.so.conf.d/imagick.conf && ldconfig \
    && chmod +x /usr/local/bin/php-boot \
    # 修改配置文件
    && sed -i "s/;extension=pdo_mysql/extension=pdo_mysql/g" ${PHP_ROOTDIR}/php.ini \
    && cp -v ${PHP_ROOTDIR}/etc/php-fpm.conf.default ${PHP_ROOTDIR}/etc/php-fpm.conf \
    && sed -i "s/;pid = run\/php-fpm.pid/pid = run\/php-fpm.pid/g" ${PHP_ROOTDIR}/etc/php-fpm.conf \
    && cp ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf.default ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf \
    && sed -i "s/listen = 127.0.0.1:9000/listen = 0.0.0.0:9000/g" ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf \
    && sed -i "s/user = nobody/user = www-data/g" ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf \
    && sed -i "s/group = nobody/group = www-data/g" ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf \
    # 安装运行时依赖
    && apt-get update && apt-get install -y --no-install-recommends \
        tzdata curl ca-certificates \
        libssl1.1 \
        libxml2 \
        libpng16-16 libjpeg62 libwebp6 libfreetype6 \
        libzip5 libsodium23 libonig5 \
        libreadline8 libsqlite3-0 zlib1g \
        libxpm4 libxslt1.1 \
        imagemagick \
        libdjvulibre21 libopenexr24 libwebpdemux2 \
    # 设置时区
    && cp /usr/share/zoneinfo/${timezone} /etc/localtime \
    && echo "${timezone}" > /etc/timezone \
    # 清理构建产物
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/*

WORKDIR /www
EXPOSE 9000
ENTRYPOINT ["php-boot"]
