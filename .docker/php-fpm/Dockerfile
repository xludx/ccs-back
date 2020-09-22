ARG TARGET_PHP_VERSION=7.3
FROM php:${TARGET_PHP_VERSION}-fpm

ARG SERVICE_DIR="./php-fpm"
COPY ./.docker/.shared/scripts/ /tmp/scripts/
COPY ./.docker/php-fpm/scripts/docker-entrypoint/run.sh /tmp/scripts/docker-entrypoint/run.sh

#COPY ./.shared/scripts/ /tmp/scripts/
RUN chmod +x -R /tmp/scripts/

# set timezone
ARG TZ=UTC
RUN /tmp/scripts/set_timezone.sh ${TZ}

# add users
ARG APP_USER=www-data
ARG APP_GROUP=www-data
ARG APP_USER_ID=1000
ARG APP_GROUP_ID=1000

RUN /tmp/scripts/create_user.sh ${APP_USER} ${APP_GROUP} ${APP_USER_ID} ${APP_GROUP_ID}

RUN /tmp/scripts/install_php_extensions.sh

RUN /tmp/scripts/install_software.sh

# php config
COPY ./.docker/.shared/config/php/conf.d/*  /usr/local/etc/php/conf.d/

# php-fpm pool config
COPY ./.docker/php-fpm/php-fpm.d/* /usr/local/etc/php-fpm.d
RUN /tmp/scripts/modify_config.sh /usr/local/etc/php-fpm.d/zz-app.conf "__APP_USER" "${APP_USER}"
RUN /tmp/scripts/modify_config.sh /usr/local/etc/php-fpm.d/zz-app.conf "__APP_GROUP" "${APP_GROUP}"

# workdir
ARG APP_CODE_PATH="/var/www/current"
WORKDIR "$APP_CODE_PATH"

# entrypoint
RUN mkdir -p /bin/docker-entrypoint/
RUN cp /tmp/scripts/docker-entrypoint/* /bin/docker-entrypoint/
RUN chmod +x -R /bin/docker-entrypoint/

RUN /tmp/scripts/cleanup.sh

# Install Composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

ENTRYPOINT ["/bin/docker-entrypoint/run.sh", "php-fpm"]
EXPOSE 9000

