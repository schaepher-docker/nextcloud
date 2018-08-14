FROM php:7.2-fpm-alpine3.7

# cron.sh dependencies
RUN rm /var/spool/cron/crontabs/root \
    && echo '*/15 * * * * php -f /var/www/html/cron.php' > /var/spool/cron/crontabs/www-data

# install the PHP extensions we need
# see https://docs.nextcloud.com/server/12/admin_manual/installation/source_installation.html
RUN apk add --no-cache --virtual .build-deps \
            $PHPIZE_DEPS \
            autoconf \
            freetype-dev \
            icu-dev \
            libjpeg-turbo-dev \
            libmcrypt-dev \
            libpng-dev \
            libmemcached-dev \
            libxml2-dev \
            openldap-dev \
            pcre-dev \
            postgresql-dev \
     && docker-php-ext-configure gd --with-freetype-dir=/usr --with-png-dir=/usr --with-jpeg-dir=/usr \ 
     && docker-php-ext-configure ldap \
     && docker-php-ext-install \
            exif \
            gd \
            intl \
            ldap \
            opcache \
            pcntl \
            pdo_mysql \
            pdo_pgsql \
            zip \
    && pecl install redis-4.1.1 \
    && docker-php-ext-enable redis \
    && runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )" \
    && apk add --virtual .nextcloud-phpext-rundeps $runDeps \
    && apk del .build-deps

# set recommended PHP.ini settings
# see https://docs.nextcloud.com/server/12/admin_manual/configuration_server/server_tuning.html#enable-php-opcache
RUN echo $'opcache.enable=1\n\
opcache.enable_cli=1\n\
opcache.interned_strings_buffer=8\n\
opcache.max_accelerated_files=10000\n\
opcache.memory_consumption=128\n\
opcache.save_comments=1\n\
opcache.revalidate_freq=1' > /usr/local/etc/php/conf.d/opcache-recommended.ini \
    && mkdir /var/www/data 
    
ENV NEXTCLOUD_VERSION 13.0.5

RUN apk add --no-cache --virtual .fetch-deps \
        bzip2 \
        gnupg \
    && curl -fsSL -o nextcloud.tar.bz2 \
        "https://download.nextcloud.com/server/releases/nextcloud-${NEXTCLOUD_VERSION}.tar.bz2" \
    && curl -fsSL -o nextcloud.tar.bz2.asc \
        "https://download.nextcloud.com/server/releases/nextcloud-${NEXTCLOUD_VERSION}.tar.bz2.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
# gpg key from https://nextcloud.com/nextcloud.asc
    && GPG_KEYS='28806A878AE423A28372792ED75899B9A724937A' \
    && ( gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEYS" \
            || gpg --keyserver hkps://hkps.pool.sks-keyservers.net --recv-keys "$GPG_KEYS" ) \
    && gpg --batch --verify nextcloud.tar.bz2.asc nextcloud.tar.bz2 \
    && tar -xjf nextcloud.tar.bz2 -C /var/www/html/ \
    && gpgconf --kill all \
    && rm -r "$GNUPGHOME" nextcloud.tar.bz2.asc nextcloud.tar.bz2 \
    && rm -rf /var/www/html/nextcloud/updater \
    && mkdir -p /var/www/html/nextcloud/data \
    && mkdir -p /var/www/html/nextcloud/custom_apps \
    && chmod +x /var/www/html/nextcloud/occ \
    && apk del .fetch-deps

COPY config/* /var/www/html/nextcloud/config/
COPY *.sh /

CMD ["php-fpm"]