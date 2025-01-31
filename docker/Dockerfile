FROM alpine:3.18 as frontend

RUN apk --no-cache add ca-certificates \
  && apk --no-cache add \
  git \
  yarn 

RUN mkdir /frontend
RUN git clone https://github.com/Secretowo/strichliste-web-frontend /frontend
WORKDIR /frontend
RUN yarn install
RUN NODE_OPTIONS=--openssl-legacy-provider yarn build

FROM composer as backend

RUN mkdir /backend
RUN git clone https://github.com/Secretowo/strichliste-backend /backend
WORKDIR /backend
RUN composer update
RUN composer install --no-dev --no-interaction --optimize-autoloader



FROM alpine:3.18

RUN apk --no-cache add ca-certificates \
  && apk --no-cache add \
  curl \
  php82 \
  php82-session \
  php82-ctype \
  php82-tokenizer \
  php82-iconv \
  php82-mbstring \
  php82-xml \
  php82-json \
  php82-dom \
  php82-pdo_mysql \
  php82-fpm \
  nginx \
  bash \
  mysql-client \
  yarn

COPY --from=backend /backend source
COPY --from=frontend /frontend/build /source/public

COPY /docker/entrypoint.sh /source/entrypoint.sh
RUN chmod +x /source/entrypoint.sh

RUN ln -s /usr/bin/php82 /usr/bin/php

RUN adduser -u 82 -D -S -G www-data www-data
RUN chown -R www-data:www-data /source
RUN chown -R www-data:www-data /var/lib/nginx
RUN chown -R www-data:www-data /var/log/nginx
RUN chown -R www-data:www-data /var/log/php82

USER www-data

COPY ./config/php-fpm.conf /etc/php82/php-fpm.conf
COPY ./config/www.conf /etc/php82/php-fpm.d/www.conf
COPY ./config/nginx.conf /etc/nginx/nginx.conf
COPY ./config/default.conf /etc/nginx/conf.d/default.conf

VOLUME /source/var

WORKDIR /source/public
EXPOSE 8080

ENTRYPOINT ["/source/entrypoint.sh"]
CMD nginx && php-fpm82