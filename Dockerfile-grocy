FROM node AS builder
ARG     version=latest

RUN     apt-get update && \
        apt-get install -y jq && \
        mkdir /www && \
        curl -s -L "$( \
            curl -s https://api.github.com/repos/grocy/grocy/releases/$( \
                [ "$version" = "latest" ] && \
                echo latest || \
                echo "tags/$version" \
            ) | jq -r '.tarball_url' \
        )" | tar zxf - -C /www --strip-components=1 && \
        cd /www/ && \
        cp /www/config-dist.php /www/data/config.php && \
        yarn install && \
        mkdir /www/data/viewcache
        

FROM php:fpm
LABEL maintainer="Stathis Stamoulis <sstamoulis2010@gmail.com>"

COPY --from=composer /usr/bin/composer /usr/bin/composer
COPY --from=builder --chown=www-data:www-data /www/ /www/
RUN     apt-get update && \
        apt-get install -y libfreetype6-dev libjpeg62-turbo-dev libpng-dev unzip && \
        docker-php-ext-configure gd \
            --with-freetype \
            --with-jpeg && \
        NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) && \
        docker-php-ext-install -j${NPROC} gd && \
        composer install -n --working-dir=/www --no-cache && \
        chown -R www-data:www-data /www && \
    # Set environments
        sed -i "s|;*daemonize\s*=\s*yes|daemonize = no|g" /usr/local/etc/php-fpm.conf && \
        sed -i "s|;*listen\s*=\s*127.0.0.1:9000|listen = 9000|g" /usr/local/etc/php-fpm.conf && \
        sed -i "s|;*listen\s*=\s*/||g" /usr/local/etc/php-fpm.conf && \
        sed -i "s|;*chdir\s*=\s*/var/www|chdir = /www|g" /usr/local/etc/php-fpm.d/www.conf

# Set Workdir
WORKDIR /www/public

# Expose volumes
VOLUME ["/www"]

# Expose ports
EXPOSE 9000
