# builder
FROM golang:alpine AS builder
LABEL maintainer="wulabing <wulabing@gmail.com>"
LABEL version="0.0.29"


#ENV GOPROXY=https://goproxy.cn,direct
WORKDIR /app

RUN apk add --no-cache --no-scripts git curl jq && \ 
    LATEST_VERSION_TAG=$(curl -s https://api.github.com/repos/XTLS/Xray-core/releases | jq -r '.[0].tag_name') && \
    git clone https://github.com/XTLS/Xray-core.git . && \
    git checkout $LATEST_VERSION_TAG && \
    go mod download -x && \
    go build -v -o xray /app/main/

# runner
FROM alpine:latest AS runner


ENV UUID=""
ENV DEST=""
ENV SERVERNAMES=""
ENV PRIVATEKEY=""
ENV SHORT_IDS=""
ENV SNI_FILTER="true"
ENV SNI_FILTER_BACKEND_PORT="65433"
ENV NETWORK=""
ENV INTERNAL_PORT=""
ENV HOSTMODE_PORT=""
ENV TZ=Asia/Shanghai

WORKDIR /

COPY ./entrypoint.sh /
COPY ./config.json /

COPY --from=builder /app/xray /

RUN apk add --no-cache --no-scripts tzdata ca-certificates jq curl libqrencode-tools unzip && \
    mkdir -p /var/log/xray && \
    wget -O /geosite.dat https://github.com/v2fly/domain-list-community/releases/latest/download/dlc.dat && \
    wget -O /geoip.dat https://github.com/v2fly/geoip/releases/latest/download/geoip.dat && \
        mkdir -p /tmp/sni-filter && \
        curl -L -o /tmp/sni-filter.zip https://github.com/oldfriendme/REALITY-sni-filter/releases/download/v0.2/autobuild.zip && \
        unzip -q /tmp/sni-filter.zip -d /tmp/sni-filter && \
        ARCH="$(uname -m)" && \
        if [ "$ARCH" = "x86_64" ]; then \
            mv /tmp/sni-filter/sni-filter-amd64 /usr/bin/sni-filter; \
        elif [ "$ARCH" = "i386" ] || [ "$ARCH" = "i686" ]; then \
            mv /tmp/sni-filter/sni-filter-i386 /usr/bin/sni-filter; \
        elif [ "$ARCH" = "aarch64" ]; then \
            mv /tmp/sni-filter/sni-filter-arm64 /usr/bin/sni-filter; \
        else \
            echo "Unsupported arch for sni-filter: $ARCH"; \
            exit 1; \
        fi && \
        chmod +x /usr/bin/sni-filter && \
        rm -rf /tmp/sni-filter /tmp/sni-filter.zip && \
    chmod +x /entrypoint.sh


ENTRYPOINT ["./entrypoint.sh"]
EXPOSE 443
