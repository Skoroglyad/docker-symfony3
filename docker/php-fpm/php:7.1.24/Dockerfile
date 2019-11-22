# PHP Dockerfile

FROM composer:1.5.1 AS composer
FROM php:7.1.24-fpm

RUN apt-get update && apt-get install -y \
      acl \
      libfreetype6-dev \
      libjpeg62-turbo-dev \
      libpng-dev \
      libbz2-dev \
      libicu-dev \
      libzip-dev \
      zip \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-install bz2 \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && apt-get install jpegoptim \
    && docker-php-ext-install intl \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install opcache \
    && docker-php-ext-configure zip --with-libzip \
    && docker-php-ext-install zip \
    && docker-php-ext-install exif \
    && pecl install xdebug-2.5.0 \
    && docker-php-ext-enable xdebug \
    && apt-get install -y vim;

# copy the Composer PHAR from the Composer image into the PHP image
COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN echo 'memory_limit = 2048M' >> /usr/local/etc/php/conf.d/docker-php-memlimit.ini

# Create user codekeeper, and use it as owner of the code
RUN useradd -ms /bin/bash codekeeper

RUN mkdir -p /code
WORKDIR /code
RUN chown codekeeper:codekeeper .

USER codekeeper

# Install composer packages (by starting with coping only composer file, to make use of docker layering feature)
COPY symfony/composer.json symfony/composer.lock ./
RUN composer install --prefer-dist --no-scripts --no-dev --no-autoloader

# Now, copy the code
COPY --chown=codekeeper:codekeeper symfony /code
WORKDIR /code

# Remove execute permissions from all files in (/code), but add it only for directories (to allow traversal)
USER root
#RUN chmod -R -x+X .


# Give permissions to write to var
RUN setfacl -dR -m u:www-data:rwX -m u:codekeeper:rwX var
RUN setfacl -R -m u:www-data:rwX -m u:codekeeper:rwX var


CMD ["sh", "-c", "(setfacl -R -m u:www-data:rwX -m u:codekeeper:rwX var) && (setfacl -dR -m u:www-data:rwX -m u:codekeeper:rwX var) && php-fpm"]