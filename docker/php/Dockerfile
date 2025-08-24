FROM php:8.3-fpm-alpine


# Install dependencies
RUN apk add --no-cache bash git icu-dev libzip-dev libpng-dev libjpeg-turbo-dev freetype-dev libxml2-dev oniguruma-dev supervisor curl


# PHP extensions
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
&& docker-php-ext-install -j$(nproc) intl pdo_mysql zip gd opcache
RUN pecl install redis && docker-php-ext-enable redis


# Opcache production
RUN { \
echo "opcache.enable=1"; \
echo "opcache.enable_cli=0"; \
echo "opcache.memory_consumption=256"; \
echo "opcache.max_accelerated_files=30000"; \
echo "opcache.validate_timestamps=0"; \
} > /usr/local/etc/php/conf.d/opcache.ini


# Custom PHP config
COPY php.ini /usr/local/etc/php/conf.d/custom.ini
COPY www.conf /usr/local/etc/php-fpm.d/www.conf


# App code
WORKDIR /var/www/html
COPY src/ .
RUN chown -R www-data:www-data storage bootstrap/cache


# Supervisord for queue
COPY supervisord.conf /etc/supervisord.conf


# Run container
CMD php artisan config:cache \
&& php-fpm -D \
&& /usr/bin/supervisord -c /etc/supervisord.conf -n