FROM alpine:3.19

ADD https://github.com/miconda/sipexer/releases/download/v1.1.0/sipexer-v1.1.0-linux-amd64.tar.gz /usr/bin/sipexer

RUN    wget -O /etc/apk/keys/whosgonna_`uname -m`.rsa.pub https://packagecloud.io/whosgonna/`uname -m`/rsakey \
    && ALPINE_MAJMIN=$(cat /etc/os-release | grep VERSION_ID | sed -e 's/VERSION_ID=\(\d*\.\d*\).*/v\1/') \
    && echo "https://packagecloud.io/whosgonna/`uname -m`/alpine/${ALPINE_MAJMIN}/main" >> /etc/apk/repositories \
    && apk add --no-cache kamailio kamailio-mysql kamailio-extras  \
                          kamailio-http_async kamailio-utils sipp sngrep \
    && chmod 755 /usr/bin/sipexer

ENTRYPOINT ["kamailio", "-dDDE"]

