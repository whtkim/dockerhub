FROM ubuntu:20.04

# 参数设置
ARG timezone="Asia/Shanghai"
ARG swoole_enable=1
ARG imagick_enable=0

# 环境变量
ENV PHP_VERSION="8.2.11" \
    REDIS_VERSION="6.0.2" \
    SWOOLE_VERSION="5.1.1" \
    IMAGICK_VERSION="3.7.0" \
    PHP_ROOTDIR="/usr/local/php" \
    TIMEZONE=${timezone}

ENV PATH="${PATH}:${PHP_ROOTDIR}/bin:${PHP_ROOTDIR}/sbin"

ENV PHPIZE_DEPS gcc g++ make autoconf file pkg-config re2c dpkg-dev libc-dev

ENV DEBIAN_FRONTEND="noninteractive"

COPY script/php-boot /usr/local/bin/php-boot

COPY script/ext-build /usr/local/bin/php-ext-build

RUN apt-get update \
 && apt-get install -y --no-install-recommends tzdata curl openssl ca-certificates \
 && cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
 && echo ${TIMEZONE} > /etc/timezone \
 # 创建目录、赋予脚本执行权限
 && if [ ! -d /www ]; then mkdir /www; fi \
 && if [ ! -d $PHP_ROOTDIR/conf.d ]; then mkdir -p $PHP_ROOTDIR/conf.d; fi \
 && chmod +x /usr/local/bin/php-boot \
 && chmod +x /usr/local/bin/php-ext-build

RUN apt-get install -y --no-install-recommends ${PHPIZE_DEPS} libsodium-dev libxslt-dev libxml2-dev libsqlite3-dev \
    libssl-dev libreadline-dev zlib1g-dev libcurl4-openssl-dev libonig-dev libzip-dev \
    libpng-dev libjpeg-dev libwebp-dev libfreetype-dev \
 && curl -o /tmp/php-${PHP_VERSION}.tar.gz https://www.php.net/distributions/php-${PHP_VERSION}.tar.gz \
 && tar -xvf /tmp/php-${PHP_VERSION}.tar.gz -C /tmp && cd /tmp/php-${PHP_VERSION} \
 && ./configure --prefix=${PHP_ROOTDIR} \
        --with-config-file-path=${PHP_ROOTDIR} \
        --with-config-file-scan-dir=${PHP_ROOTDIR}/conf.d \
        --with-pdo-mysql=shared,mysqlnd --enable-shared --enable-mysqlnd \
        --enable-gd --with-webp --with-jpeg --with-freetype \
        --with-openssl \
        --with-zlib \
        --with-curl \
        --with-pear \
        --with-zip \
        --with-readline \
        --with-mhash \
        --with-sodium \
        --with-xsl \
        --enable-fpm \
        --enable-mbstring \
        --enable-pcntl \
        --enable-sockets \
        --enable-session \
        --enable-bcmath \
        --enable-intl \
        --enable-soap \
 && make && make install \
 #二进制压缩 \
 && find ${PHP_ROOTDIR}/bin/ -type f -exec strip {} \; \
 && find ${PHP_ROOTDIR}/sbin/ -type f -exec strip {} \; \
 #php配置
 && cp -v php.ini-* ${PHP_ROOTDIR}/ \
 && cp -v php.ini-production ${PHP_ROOTDIR}/php.ini \
 && sed -i "s/;extension=pdo_mysql/extension=pdo_mysql/g" ${PHP_ROOTDIR}/php.ini \
 && cp -v ${PHP_ROOTDIR}/etc/php-fpm.conf.default ${PHP_ROOTDIR}/etc/php-fpm.conf \
 && sed -i "s/;pid = run\/php-fpm.pid/pid = run\/php-fpm.pid/g" ${PHP_ROOTDIR}/etc/php-fpm.conf \
 && cp ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf.default ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf \
 && sed -i "s/listen = 127.0.0.1:9000/listen = 0.0.0.0:9000/g" ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf \
 && sed -i "s/user = nobody/user = www-data/g" ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf \
 && sed -i "s/group = nobody/group = www-data/g" ${PHP_ROOTDIR}/etc/php-fpm.d/www.conf \
 #composer安装
 && curl -o /usr/local/bin/composer https://mirrors.aliyun.com/composer/composer.phar \
 && chmod +x /usr/local/bin/composer \
 #清理工作
 && rm -rf /tmp/*

RUN . /usr/local/bin/php-ext-build \
 #redis扩展
 && pecl install https://pecl.php.net/get/redis-${REDIS_VERSION}.tgz \
 && echo "extension=redis.so" >> ${PHP_ROOTDIR}/conf.d/extension.ini \
 #其他扩展
 && if [ "$swoole_enable" = "1" ]; then buildSwoole; fi \
 && if [ "$imagick_enable" = "1" ]; then buildImagick; fi

RUN apt-get autoremove -y && apt-get autoclean -y && apt-get clean -y \
 && rm -rf /tmp/* /var/tmp/* /var/lib/apt/lists/* /var/cache/apt/* \
 && rm -rf /usr/share/{doc,man} \
 && find /var/log -type f -delete

EXPOSE 9000

WORKDIR ${PHP_ROOTDIR}

ENTRYPOINT ["php-boot"]
