FROM php:8.2-fpm-alpine as app

# Useful PHP extension installer image, copy binary into your container
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

# RUN docker-php-ext-install pdo pdo_mysql
# install php extensions
# exit on errors, exit on unset variables, print every command as it is executed
RUN set -eux;  \
    install-php-extensions pdo pdo_mysql;

# allow super user - set this if you use Composer as a
# super user at all times line in docker containers
ENV COMPOSER_ALLOW_SUPERUSER 1

# obtain composer using multi-stage build
# https://docs.docker.com/build/building/multi-stage/
COPY --from=composer:2.5.4 /usr/bin/composer /usr/bin/composer

# Here we are only copying only composer.json and composer.lock
# right before doing composer install
# This is enough to take advantage of docker cache and composer install will be
# executed only when composer.json or composer.lock changes
# https://medium.com/@softius/faster-docker-builds-with-composer-install-b4d2b15d0fff
COPY ./app/composer.* ./

# install
RUN composer install --prefer-dist --no-scripts --no-dev --no-progress --no-interaction

# copy application files to the working directory
COPY ./app .

# run composer dump-autoload --optimize
RUN composer dump-autoload --optimize

FROM app as app_dev

ENV XDEBUG_MODE=off

COPY ./php/conf.d/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN set -eux;  \
    install-php-extensions xdebug;