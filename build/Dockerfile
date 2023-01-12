FROM php:8.0-alpine

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
        php8 php8-fpm \
        php8-ctype php8-curl php8-dom php8-fileinfo php8-gd \
        php8-iconv php8-intl php8-json php8-mbstring \
        php8-mysqli php8-openssl php8-pdo_mysql \
        php8-session php8-simplexml php8-tokenizer php8-xml php8-xmlreader php8-xmlwriter \
        php8-zip php8-zlib php8-phar php8-opcache php8-sodium git \
        gnu-libiconv npm jq make g++ composer
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

VOLUME $DOCKER_PROJECT_DIR

ADD $BUILD_BIN_DIR/* /bin/

ENTRYPOINT /bin/entrypoint $DOCKER_PROJECT_DIR

CMD /bin/warm-up

HEALTHCHECK --timeout=10s CMD curl --silent --fail $FPM_PING_URL
