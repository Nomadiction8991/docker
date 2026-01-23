FROM php:8.3-apache

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    libzip-dev \
    unzip \
    curl \
    git \
    && docker-php-ext-install zip mysqli pdo pdo_mysql \
    && a2enmod rewrite \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/html
COPY . /var/www/html

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

RUN composer install --no-interaction --prefer-dist --no-progress

COPY Dockerfiles/apache.conf /etc/apache2/sites-available/000-default.conf
COPY Dockerfiles/php.ini /usr/local/etc/php/conf.d/dev.ini

EXPOSE 80
