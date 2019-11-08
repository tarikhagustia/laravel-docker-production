#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#

ARG PHP_VERSION=7.3
FROM php:${PHP_VERSION}-alpine

LABEL maintainer="Tarikh Agustia <agustia.tarikh150@gmail.com>"

RUN apk --update add wget \
  curl \
  git \
  build-base \
  libmemcached-dev \
  libmcrypt-dev \
  libxml2-dev \
  pcre-dev \
  zlib-dev \
  autoconf \
  cyrus-sasl-dev \
  libgsasl-dev \
  supervisor

RUN docker-php-ext-install mysqli mbstring pdo pdo_mysql tokenizer xml pcntl
RUN pecl channel-update pecl.php.net && pecl install memcached mcrypt-1.0.1 mongodb && docker-php-ext-enable memcached mongodb
RUN curl -o /usr/local/bin/composer https://getcomposer.org/composer.phar && chmod a+x /usr/local/bin/composer

# Install PostgreSQL drivers:
RUN apk --update add postgresql-dev \
            && docker-php-ext-install pdo_pgsql

# Install ZipArchive:
RUN apk --update add libzip-dev && \
        docker-php-ext-configure zip --with-libzip && \
        docker-php-ext-install zip

# Install MySQL Client:
RUN apk --update add mysql-client

# Install Redis package:
RUN printf "\n" | pecl install -o -f redis \
        &&  rm -rf /tmp/pear \
        &&  docker-php-ext-enable redis

#
#--------------------------------------------------------------------------
# Optional Supervisord Configuration
#--------------------------------------------------------------------------
#
# Modify the ./supervisor.conf file to match your App's requirements.
# Make sure you rebuild your container with every change.
#


COPY deploy/php-worker/supervisord.conf /etc/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisord.conf"]

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# If you need to modify this image, feel free to do it right here.
#
    # -- Your awesome modifications go here -- #

# Installing Composer
WORKDIR /var/www

COPY app app/
COPY bootstrap bootstrap/
COPY config config/
COPY database database/
COPY public public/
COPY resources resources/
COPY routes routes/
COPY storage storage/
COPY tests tests/

COPY composer.json ./
COPY composer.lock ./
COPY .env ./
COPY artisan ./
COPY package.json ./
COPY packages packages/

RUN composer global require hirak/prestissimo && \
    composer install --prefer-dist && \
    composer clear-cache

RUN php artisan config:cache && php artisan view:cache

#
#--------------------------------------------------------------------------
# Check PHP version
#--------------------------------------------------------------------------
#

RUN php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

# Clean up
RUN rm /var/cache/apk/* \
    && mkdir -p /var/www

WORKDIR /etc/supervisor/conf.d/
