FROM alpine:3.13

ARG PROJECT_DIR
ARG BUILD_DIR
ARG DOCKER_PROJECT_DIR
ARG DOCKER_USER
ARG COMPOSER_HOME
ARG HTTP_HOST
ARG HTTP_PORT

ENV BUILD_BIN_DIR=$BUILD_DIR/bin
ENV LD_PRELOAD=/usr/lib/preloadable_libiconv.so
ENV FPM_PING_URL=http://127.0.0.1:$HTTP_HOST/fpm-ping

RUN apk --no-cache add \
        nginx supervisor curl zip rsync xz coreutils \
        php7 php7-fpm \
        php7-ctype php7-curl php7-dom php7-fileinfo php7-gd \
        php7-iconv php7-intl php7-json php7-mbstring \
        php7-mysqli php7-openssl php7-pdo_mysql \
        php7-session php7-simplexml php7-tokenizer php7-xml php7-xmlreader php7-xmlwriter \
        php7-zip php7-zlib php7-phar php7-opcache php7-sodium git \
        gnu-libiconv npm jq make g++ \
    && adduser -u 1000 -D -h$DOCKER_PROJECT_DIR $DOCKER_USER $DOCKER_USER \
    && rm /etc/nginx/conf.d/default.conf \
    && mkdir -p $COMPOSER_HOME

# Copy system configs
COPY $BUILD_DIR/config/etc /etc

# Make sure files/folders needed by the processes are accessible when they run under the sw6
RUN mkdir -p /var/{lib,tmp,log}/nginx \
    && chown -R $DOCKER_USER.$DOCKER_USER /run /var/{lib,tmp,log}/nginx \
    && chown -R $DOCKER_USER.$DOCKER_USER $COMPOSER_HOME

WORKDIR $DOCKER_PROJECT_DIR

USER $DOCKER_USER

ADD --chown=$DOCKER_USER $PROJECT_DIR $DOCKER_PROJECT_DIR

EXPOSE $HTTP_PORT

VOLUME $DOCKER_PROJECT_DIR

ADD $BUILD_BIN_DIR/* /bin/

ENTRYPOINT /bin/entrypoint $DOCKER_PROJECT_DIR

CMD /bin/warm-up

HEALTHCHECK --timeout=10s CMD curl --silent --fail $FPM_PING_URL
