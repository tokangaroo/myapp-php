# ==================================================
# PHP 8.4.26-FPM 镜像构建
# ==================================================

# --------------------------------------------------
# 构建参数
# --------------------------------------------------

# 构建参数指令
ARG PHP_VERSION=8.4.26-fpm-bookworm

# 基于基础镜像
FROM php:${PHP_VERSION}

# 构建时默认时区
ARG TZ=Asia/Shanghai

# 运行时默认时区
ENV TZ=${TZ}

# --------------------------------------------------
# 安装系统依赖包
# --------------------------------------------------

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        git curl wget unzip zip vim tzdata ca-certificates libfcgi-bin \
        libfreetype6-dev libjpeg62-turbo-dev libpng-dev libwebp-dev libavif-dev libmagickwand-dev libmagickcore-dev \
        libzip-dev zlib1g-dev libssl-dev \
        libicu-dev libonig-dev libxml2-dev libxslt1-dev gettext \
        libpq-dev libgmp-dev \
        libcurl4-openssl-dev libmemcached-dev libyaml-dev; \
    rm -rf /var/lib/apt/lists/*

# --------------------------------------------------
# 配置GD扩展
# --------------------------------------------------

RUN set -eux; \
    docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg \
        --with-webp \
        --with-avif

# --------------------------------------------------
# 安装内置扩展
# --------------------------------------------------

RUN set -eux; \
    docker-php-ext-install -j2 \
        mysqli pdo_mysql pdo_pgsql pgsql \
        opcache \
        gd exif \
        zip bcmath gmp \
        intl mbstring gettext soap xsl \
        curl \
        pcntl sockets sysvmsg sysvsem sysvshm shmop calendar

# --------------------------------------------------
# 安装PECL扩展
# --------------------------------------------------

RUN set -eux; \
    pecl channel-update pecl.php.net; \
    pecl install igbinary redis memcached mongodb imagick apcu yaml; \
    docker-php-ext-enable igbinary redis memcached mongodb imagick apcu yaml; \
    rm -rf /tmp/pear

# --------------------------------------------------
# 安装Composer设置变量
# --------------------------------------------------

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_NO_INTERACTION=1

# --------------------------------------------------
# PHP基础配置
# --------------------------------------------------

RUN set -eux; \
    cp "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"; \
    echo "date.timezone = ${TZ}" > "$PHP_INI_DIR/conf.d/timezone.ini"

# --------------------------------------------------
# 工作目录与启动
# --------------------------------------------------

WORKDIR /var/www/html

# --------------------------------------------------
# 声明监听端口
# --------------------------------------------------

EXPOSE 9000

# --------------------------------------------------
# 健康检查
# --------------------------------------------------

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD SCRIPT_NAME=/fpm-ping SCRIPT_FILENAME=/fpm-ping REQUEST_METHOD=GET \
        cgi-fcgi -bind -connect 127.0.0.1:9000 | grep -q pong || exit 1

# --------------------------------------------------
# 启动命令
# --------------------------------------------------

CMD ["php-fpm"]