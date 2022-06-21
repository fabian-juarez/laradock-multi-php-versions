FROM php:7.2-fpm-alpine

LABEL maintainer="Fabián Juárez <fabian.main@gmail.com>"

# Basic tools
RUN export TZ=America/Mexico_City
RUN apk add --no-cache tzdata
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY php7.2.ini /usr/local/etc/php/php.ini
COPY laravel.ini /usr/local/etc/php/conf.d
COPY xlaravel.pool.conf /usr/local/etc/php-fpm.d/

RUN docker-php-ext-install \
    pdo_mysql

RUN apk add --no-cache shadow && \
    usermod -u 1000 www-data && \
    apk del shadow

#####################################
# Extra extension:
#####################################

RUN apk add --no-cache libzip-dev
RUN docker-php-ext-install zip


# Install composer
ARG PROJECT_PHP72_INSTALL_COMPOSER=false
RUN if [ ${PROJECT_PHP72_INSTALL_COMPOSER} = true ]; then \
      php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
      php composer-setup.php --install-dir=/usr/local/bin --filename=composer --2 \
    ;fi

# Install apcu
RUN docker-php-source extract \
    && apk add --no-cache --virtual .phpize-deps-configure $PHPIZE_DEPS \
    && pecl install apcu \
    && docker-php-ext-enable apcu \
    && apk del .phpize-deps-configure \
    && docker-php-source delete

# Install GD
RUN apk add --no-cache \
    freetype \
    libpng \
    libjpeg-turbo

RUN apk add --no-cache --virtual gd-dependencies \
  freetype-dev \
  libjpeg-turbo-dev \
  libpng-dev \
  && docker-php-ext-install gd \
  && docker-php-ext-configure gd \
  --with-freetype-dir=/usr/include/ \
  --with-jpeg-dir=/usr/include/ \
  --with-png-dir=/usr/include/ \
  && apk del gd-dependencies

# Fix to iconv
# See: https://github.com/docker-library/php/issues/240#issuecomment-305038173
RUN apk add --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing --allow-untrusted gnu-libiconv
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

# Install exif
RUN docker-php-ext-install exif

####################################

CMD ["php-fpm"]

EXPOSE 80 9000
