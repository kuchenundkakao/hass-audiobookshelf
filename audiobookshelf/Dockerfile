ARG BUILD_FROM
## STAGE 0: Build Client ###

ARG NUSQLITE3_DIR="/usr/local/lib/nusqlite3"
ARG NUSQLITE3_PATH="${NUSQLITE3_DIR}/libnusqlite3.so"

FROM node:20-alpine AS build
ADD https://github.com/advplyr/audiobookshelf/archive/refs/tags/v2.24.0.tar.gz /
RUN cd / && \
    tar xvf v2.24.0.tar.gz && \
    cd /audiobookshelf-2.24.0/client/ && \
    npm ci && npm cache clean --force \
    npm install -g npm@10.2.4 && \
    npm run generate && \
    ls -l /audiobookshelf-2.24.0/client

## STAGE 1: Build Server ###

FROM sandreas/tone:v0.2.5 AS tone

# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
FROM $BUILD_FROM
#FROM ghcr.io/hassio-addons/base:14.3.2

ARG NUSQLITE3_DIR
ARG BUILD_ARCH

# Execute during the build of the image
ARG TEMPIO_VERSION BUILD_ARCH
RUN \
    curl -sSLf -o /usr/bin/tempio \
    "https://github.com/home-assistant/tempio/releases/download/${TEMPIO_VERSION}/tempio_${BUILD_ARCH}"

RUN apk update && \
    apk add --no-cache --update \
    curl \
    tzdata \
    ffmpeg \
    make \
    python3 \
    g++ \
    npm

ENV NODE_ENV=production

RUN case "$BUILD_ARCH" in \
  "amd64") \
  curl -L -o /tmp/library.zip "https://github.com/mikiher/nunicode-binaries/releases/download/v1.2/libnusqlite3-linux-musl-x64.zip" ;; \
  "aarch64") \
  curl -L -o /tmp/library.zip "https://github.com/mikiher/nunicode-sqlite/releases/download/v1.2/libnusqlite3-linux-musl-arm64.zip" ;; \
  *) echo "Unsupported Home Assistant architecture for libnusqlite3: $BUILD_ARCH" && exit 1 ;; \
  esac && \
  unzip /tmp/library.zip -d $NUSQLITE3_DIR && \
  rm /tmp/library.zip
  
COPY rootfs /

COPY --from=tone /usr/local/bin/tone /usr/local/bin/
COPY --from=build /audiobookshelf-2.24.0/client/dist /client/dist
COPY --from=build /audiobookshelf-2.24.0/server /server
COPY --from=build /audiobookshelf-2.24.0/package* /
COPY --from=build /audiobookshelf-2.24.0/index.js /

RUN npm ci --only=production

RUN apk del make python3 g++

EXPOSE 8081

ENV SOURCE="docker"
ENV NUSQLITE3_DIR=${NUSQLITE3_DIR}
ENV NUSQLITE3_PATH=${NUSQLITE3_PATH}

#CMD ["node", "index.js"]
