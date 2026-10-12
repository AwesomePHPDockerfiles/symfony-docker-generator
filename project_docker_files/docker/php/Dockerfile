# syntax=docker/dockerfile:1

FROM php:8.5-fpm-trixie AS php-base

# Install PIE (PHP Installer for Extensions) https://php.github.io/pie/#docs/usage
COPY --from=ghcr.io/php/pie:bin /pie /usr/local/bin/pie

# Install composer from https://hub.docker.com/_/composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Install Symfony CLI https://symfony.com/download#step-1-install-symfony-cli
COPY --link \
    --from=ghcr.io/symfony-cli/symfony-cli:latest \
    /usr/local/bin/symfony /usr/local/bin/symfony

RUN <<EOT
#!/bin/bash
    export DEBIAN_FRONTEND="noninteractive"

    set -eux

    apt-get update

    # Install permanent dependencies
    PERM_DEPS=(
        bash
        git
        libbz2-1.0
        libicu76
        libpq5
        libxml2
        libzip5
        unzip
    )
    apt-get install --yes --no-install-recommends "${PERM_DEPS[@]}"

    # Install build-time dependencies
    BUILD_DEPS=(
        build-essential
        libavif-dev
        libbz2-dev
        libfreetype6-dev
        libicu-dev
        libjpeg-dev
        libpng-dev
        libpq-dev
        librabbitmq-dev
        libtool
        libwebp-dev
        libxml2-dev
        libxslt1-dev
        libzip-dev
    )
    apt-get install --yes --no-install-recommends "${BUILD_DEPS[@]}"

    # Install PHP extensions
    PHP_EXTS=(
        bcmath
        bz2
        gd
        intl
        pcntl
        pdo_pgsql
        soap
        sockets
        xsl
        zip
    )

    docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg \
        --with-webp \
        --with-avif

    docker-php-ext-install -j "$(nproc)" "${PHP_EXTS[@]}"

    pie install pie-extensions/igbinary:3.2.17RC1
    pie install xdebug/xdebug:3.5.3
    pie install phpredis/phpredis:6.3.0
    pie install php-amqp/php-amqp:2.2.0

    # TODO надо подумать как правильно удалить build-time зависимости, чтобы не сломать работу с расширениями, возможно так правильнее
    # apt-mark auto "${BUILD_DEPS[@]}"
    # # install exts...
    # apt-get autoremove -y --purge

    # Remove build-time dependencies
    # apt-get remove --yes --purge --auto-remove "${BUILD_DEPS[@]}"
    # docker-php-source delete

    # Clean apt cache
    apt-get clean
    rm -rf /var/lib/apt/lists/*
EOT

# Create unprivileged user
RUN groupadd appuser \
    && useradd -m -g appuser -s /bin/bash appuser
COPY --chown=appuser:appuser docker/.bashrc /home/appuser/

# Allow Composer to use all available RAM while installing packages
ENV COMPOSER_MEMORY_LIMIT="-1"

WORKDIR /var/www/


FROM php-base AS php-fpm

USER appuser:appuser

CMD ["php-fpm"]
