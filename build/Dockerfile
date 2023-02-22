FROM php:8.2-cli-alpine3.17

ARG PROJECT_DIR
ARG BUILD_DIR
ARG DOCKER_PROJECT_DIR
ARG DOCKER_USER
ARG DOCKER_USER_ID
ARG COMPOSER_HOME
ARG HTTP_HOST
ARG HTTP_PORT

ENV BUILD_BIN_DIR=$BUILD_DIR/bin
ENV LD_PRELOAD=/usr/lib/preloadable_libiconv.so
ENV FPM_PING_URL=http://127.0.0.1:$HTTP_HOST/fpm-ping

RUN apk --no-cache add \
        nginx supervisor curl zip rsync xz coreutils \
        php php-fpm \
        php-ctype php-curl php-dom php-fileinfo php-gd \
        php-iconv php-intl php-json php-mbstring \
        php-mysqli php-openssl php-pdo_mysql \
        php-session php-simplexml php-tokenizer php-xml php-xmlreader php-xmlwriter \
        php-zip php-zlib php-phar php-opcache php-sodium git \
        gnu-libiconv npm jq make g++ composer bash

RUN set -xe \
    && apk add --update \
        icu \
    && apk add --no-cache --virtual .php-deps \
        make \
    && apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        zlib-dev \
        icu-dev \
        g++ \
    && docker-php-ext-configure intl \
    && docker-php-ext-install intl \
    && docker-php-ext-enable intl \
    && { find /usr/local/lib -type f -print0 | xargs -0r strip --strip-all -p 2>/dev/null || true; } \
    && apk del .build-deps \
    && rm -rf /tmp/* /usr/local/lib/php/doc/* /var/cache/apk/*

RUN adduser -u $DOCKER_USER_ID -D -h$DOCKER_PROJECT_DIR $DOCKER_USER $DOCKER_USER
RUN docker-php-ext-install pdo_mysql
RUN mkdir -p $COMPOSER_HOME

# Copy system configs
COPY $BUILD_DIR/config/etc /etc

# Make sure files/folders needed by the processes are accessible when they run under the sw6
RUN mkdir -p /var/{lib,tmp,log}/nginx \
    && chown -R $DOCKER_USER:$DOCKER_USER /run /var/{lib,tmp,log}/nginx \
    && chown -R $DOCKER_USER:$DOCKER_USER $COMPOSER_HOME

WORKDIR $DOCKER_PROJECT_DIR

USER $DOCKER_USER

ADD --chown=$DOCKER_USER $PROJECT_DIR $DOCKER_PROJECT_DIR

EXPOSE $HTTP_PORT

EXPOSE $STOREFRONT_PROXY_PORT

VOLUME $DOCKER_PROJECT_DIR

ADD $BUILD_BIN_DIR/* /bin/

ENTRYPOINT /bin/entrypoint $DOCKER_PROJECT_DIR

CMD /bin/warm-up

HEALTHCHECK --timeout=10s CMD curl --silent --fail $FPM_PING_URL
