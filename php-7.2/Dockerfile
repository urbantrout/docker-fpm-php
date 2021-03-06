ARG     PHP_VERSION="${PHP_VERSION:-7.2.14}"
FROM	php:${PHP_VERSION}-fpm-alpine

LABEL	maintainer="https://github.com/hermsi1337"

ARG     PHPREDIS_VERSION="${PHPREDIS_VERSION:-4.2.0}"
ENV     PHPREDIS_VERSION="${PHPREDIS_VERSION}"

ADD     http://downloads3.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz /tmp/
ADD     https://github.com/phpredis/phpredis/archive/${PHPREDIS_VERSION}.tar.gz /tmp/

COPY ./locale.md /locale.md

RUN apk update && \
    \
    apk upgrade && \
    \
    docker-php-source extract && \
    \
    apk add --no-cache \
    --virtual .build-dependencies \
    $PHPIZE_DEPS \
    zlib-dev \
    cyrus-sasl-dev \
    git \
    autoconf \
    g++ \
    libtool \
    make \
    pcre-dev && \
    \
    apk add --no-cache \
    tini \
    libintl \
    icu \
    icu-dev \
    libxml2-dev \
    postgresql-dev \
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    git \
    gmp \
    gmp-dev \
    libmemcached-dev \
    imagemagick \
    imagemagick-dev \
    libssh2 \
    libssh2-dev \
    libxslt-dev && \
    \
    tar xfz /tmp/${PHPREDIS_VERSION}.tar.gz && \
    \
    mv phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis && \
    \
    git clone https://github.com/php-memcached-dev/php-memcached.git /usr/src/php/ext/memcached/ && \
    \
    docker-php-ext-configure memcached &&  \
    \
    docker-php-ext-configure gd \
    --with-freetype-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ &&  \
    \
    docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" \
    intl \
    bcmath \
    xsl \
    zip \
    soap \
    mysqli \
    pdo \
    pdo_mysql \
    pdo_pgsql \
    gmp \
    redis \
    iconv \
    gd \
    memcached &&  \
    \
    tar -xvzf /tmp/ioncube_loaders_lin_x86-64.tar.gz -C /tmp/ &&  \
    \
    mkdir -p /usr/local/php/ext/ioncube &&  \
    \
    cp  /tmp/ioncube/ioncube_loader_lin_${PHP_VERSION%.*}.so \
    /usr/local/php/ext/ioncube/ioncube_loader.so &&  \
    \
    docker-php-ext-configure opcache --enable-opcache &&  \
    \
    docker-php-ext-install opcache &&  \
    \
    pecl install \
    apcu imagick ssh2-1 &&  \
    \
    docker-php-ext-enable \
    apcu imagick ssh2 &&  \
    \
    apk del .build-dependencies &&  \
    \
    docker-php-source delete &&  \
    \
    apk --no-cache add ca-certificates wget && \
    \
    wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub && \
    \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-2.25-r0.apk && \
    \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-bin-2.25-r0.apk && \
    \
    wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/glibc-i18n-2.25-r0.apk && \
    \
    apk add glibc-bin-2.25-r0.apk glibc-i18n-2.25-r0.apk glibc-2.25-r0.apk && \
    \
    cat locale.md | xargs -i /usr/glibc-compat/bin/localedef -i {} -f UTF-8 {}.UTF-8 && \
    \
    rm -rf /tmp/* /var/cache/apk/*

# Install composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Fix OpenSSL
RUN sed -i 's/#default_bits/default_bits/' /etc/ssl/openssl.cnf

# set recommended PHP.ini settings
# https://secure.php.net/manual/en/opcache.installation.php
# https://secure.php.net/manual/en/apcu.configuration.php
# also, enable ioncube
COPY    conf.d/* /usr/local/etc/php/conf.d/

WORKDIR /var/www/html
RUN chown -R www-data:www-data .
USER www-data

CMD     ["php-fpm"]
