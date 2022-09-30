FROM alpine:latest

RUN apk add --no-cache \
    curl \
    git

ENV VERSION 0.88.1
RUN mkdir -p /usr/local/src \
    && cd /usr/local/src \

    && curl -L https://github.com/gohugoio/hugo/releases/download/v${VERSION}/hugo_${VERSION}_linux-64bit.tar.gz | tar -xz \
    && mv hugo /usr/local/bin/hugo \

    && addgroup -Sg 1000 hugo \
    && adduser -SG hugo -u 1000 -h /src hugo

WORKDIR /src

EXPOSE 1313
