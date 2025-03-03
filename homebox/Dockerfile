# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM
FROM $BUILD_FROM

ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION
ARG BASHIO_VERSION=0.16.2
ARG S6_OVERLAY_VERSION=3.1.6.2

LABEL \
    org.opencontainers.image.source=https://github.com/sysadminsmedia/homebox-ha-addon \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    org.opencontainers.image.title="Home Assistant Add-on: Homebox"\
    org.opencontainers.image.description="Installs homebox under hassio" \
    org.opencontainers.image.source="https://github.com/sysadminsmedia/homebox-ha-addon" \
    maintainer="John Scarrott <awesome@example.com>" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="John Scarrott <awesome@example.com>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}

# Execute during the build of the image
#Add nginx and create the run folder for nginx.
ADD http://luarocks.org/releases/luarocks-2.0.13.tar.gz /tmp/luarocks-2.0.13.tar.gz
RUN \
    apk update \
    && apk --no-cache add \
    nginx build-base bash curl nginx-mod-http-lua lua-dev jq \
    \
    && cd /tmp \
    && tar -xzvf luarocks-2.0.13.tar.gz \
    && cd luarocks-2.0.13/ \
    ##&& ./configure --prefix=/usr/local/openresty/luajit --with-lua=/usr/local/openresty/luajit/ --lua-suffix=jit --with-lua-include=/usr/local/openresty/luajit/include/luajit-2.1 \
    && ./configure --lua-suffix=jit \
    && make \
    && make install \
    #&& luarocks install lua-resty-http 0.15-0 \
    && mkdir -p /run/nginx \
    && if [ "${BUILD_ARCH}" = "armv7" ]; then \
    export S6_ARCH="arm"; \
    elif [ "${BUILD_ARCH}" = "i386" ]; then \
    export S6_ARCH="i686"; \
    elif [ "${BUILD_ARCH}" = "amd64" ]; then \
    export S6_ARCH="x86_64"; \
    else \
    export S6_ARCH="${BUILD_ARCH}"; \
    fi \
    \
    && echo "**** install s6 overlay ****" \
    && echo "S6_ARCH: ${S6_ARCH} S6_OVERLAY_VERSION: ${S6_OVERLAY_VERSION}" \
    && curl -L -f -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-${S6_ARCH}.tar.xz" \
    | tar Jxvf - -C / \
    && curl -L -f -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz" \
    | tar Jxvf - -C / \
    && curl -L -f -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-symlinks-arch.tar.xz" \
    | tar Jxvf - -C / \
    && curl -L -f -s "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-symlinks-noarch.tar.xz" \
    | tar Jxvf - -C / \
    && mkdir -p /usr/src/bashio \
    && curl -L -f -s "https://github.com/hassio-addons/bashio/archive/v${BASHIO_VERSION}.tar.gz" \
    | tar -xzf - --strip 1 -C /usr/src/bashio \
    && mv /usr/src/bashio/lib /usr/lib/bashio \
    && ln -s /usr/lib/bashio/bashio /usr/bin/bashio

#Copy our conf into the nginx http.d folder.
#COPY ingress.conf /etc/nginx/http.d/
COPY /rootfs /

#Launch nginx with debug options.
#RUN [ "nginx","-g","daemon off;error_log /dev/stdout debug;" ]
WORKDIR /
ENTRYPOINT ["/init"]
#ENTRYPOINT [ "/usr/bin/run_homebox" ]
#CMD [ "/data/config.yml" ]
