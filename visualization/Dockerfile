# syntax = docker/dockerfile:experimental
FROM ruby:2.6.3-alpine as builder
LABEL maintainer "mi2428 <tmiya@protonmail.ch>"

ENV BUNDLE_JOBS=4
ENV RAILS_ENV=production

WORKDIR /app

COPY sindan-visualization/. .
COPY database.yml config
COPY secrets.yml config
COPY seeds.rb db

# hadolint ignore=DL3018,DL3028
RUN apk add --no-cache --update \
      build-base  alpine-sdk tzdata shadow sudo ca-certificates \
      nodejs busybox-suid mariadb-dev mariadb-connector-c-dev libxml2-dev libxslt-dev libstdc++ \
 && gem install bundler --no-document \
 && bundle config build.nokogiri --use-system-libraries \
 && bundle install --without development test --path vendor/bundle

# hadolint ignore=SC2215
RUN --mount=type=secret,id=rails_secret,target=/run/secrets/rails_secret_key_base,required \
    --mount=type=secret,id=db_pass,target=/run/secrets/db_password,required \
    bundle exec rails assets:precompile

FROM ruby:2.6.3-alpine as production
LABEL maintainer "mi2428 <tmiya@protonmail.ch>"

ENV LANG C.UTF-8
ENV BUNDLE_JOBS=4
ENV TZ=Asia/Tokyo
ENV RAILS_ENV=production

WORKDIR /app

COPY sindan-visualization/. .
COPY database.yml config
COPY secrets.yml config
COPY seeds.rb db

# hadolint ignore=DL3018,DL3028
RUN apk add --no-cache --update \
      libxml2-dev libxslt-dev libstdc++ tzdata nodejs mariadb-connector-c ca-certificates

COPY --from=builder /app/vendor/bundle vendor/bundle
COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY --from=builder /app/public/assets /app/public/assets

EXPOSE 3000
