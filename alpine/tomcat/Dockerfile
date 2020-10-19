FROM alpine:3.12

RUN { echo '#!/bin/sh'; echo 'set -e'; echo; echo 'dirname "$(dirname "$(readlink -f "$(which javac || which java)")")"';} > /usr/local/bin/docker-java-home && chmod +x /usr/local/bin/docker-java-home

ENV JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk/jre

ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin

ENV JAVA_VERSION=8u252

ENV JAVA_ALPINE_VERSION=8.252.09-r0

RUN apk add --no-cache openjdk8-jre

ENV CATALINA_HOME=/usr/local/tomcat

ENV PATH=/usr/local/tomcat/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin

RUN mkdir -p "$CATALINA_HOME"

WORKDIR /usr/local/tomcat

ENV TOMCAT_NATIVE_LIBDIR=/usr/local/tomcat/native-jni-lib

ENV LD_LIBRARY_PATH=/usr/local/tomcat/native-jni-lib

ENV GPG_KEYS=A9C5DF4D22E99998D9875A5110C01C5A2F6059E7

ENV TOMCAT_MAJOR=8

ENV TOMCAT_VERSION=8.5.59

ENV TOMCAT_SHA512=3106fa39f1859ba31c87d245eaf8efe9c1f2f4837dc4f259c245d2c9b55d3593e97a9925e7d012d397987279965b8e24d940cfdda062d8ac9b64e41396120bd9

ENV TOMCAT_TGZ_URLS=https://www.apache.org/dyn/closer.cgi?action=download&filename=tomcat/tomcat-8/v8.5.59/bin/apache-tomcat-8.5.59.tar.gz

ENV TOMCAT_ASC_URLS=https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.59/bin/apache-tomcat-8.5.59.tar.gz.asc

ENV DOCKER_JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk

RUN set -eux; apk add --no-cache --virtual .fetch-deps gnupg ca-certificates openssl ; \
    export GNUPGHOME="$(mktemp -d)"; \
    for key in $GPG_KEYS; do gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; done; success=; \
    for url in $TOMCAT_TGZ_URLS; do if wget -O tomcat.tar.gz "$url"; then success=1; break; fi; done; 	[ -n "$success" ]; \
    echo "$TOMCAT_SHA512 *tomcat.tar.gz" | sha512sum -c -; success=; \
    for url in $TOMCAT_ASC_URLS; do if wget -O tomcat.tar.gz.asc "$url"; then success=1; break; fi; done; [ -n "$success" ]; \
    gpg --batch --verify tomcat.tar.gz.asc tomcat.tar.gz; 	tar -xvf tomcat.tar.gz --strip-components=1; \
    rm bin/*.bat; \
    rm tomcat.tar.gz*; 	command -v gpgconf && gpgconf --kill all || :; 	rm -rf "$GNUPGHOME"; \
    nativeBuildDir="$(mktemp -d)"; 	tar -xvf bin/tomcat-native.tar.gz -C "$nativeBuildDir" --strip-components=1; \
    apk add --no-cache --virtual .native-build-deps apr-dev coreutils dpkg-dev dpkg gcc libc-dev make "openjdk${JAVA_VERSION%%[-~bu]*}"="$JAVA_ALPINE_VERSION" openssl-dev ; \
    ( export CATALINA_HOME="$PWD"; cd "$nativeBuildDir/native"; gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; ./configure --build="$gnuArch" --libdir="$TOMCAT_NATIVE_LIBDIR" --prefix="$CATALINA_HOME" --with-apr="$(which apr-1-config)" --with-java-home="$(docker-java-home)" --with-ssl=yes; \
    make -j "$(nproc)"; make install; ); \
    rm -rf "$nativeBuildDir"; rm bin/tomcat-native.tar.gz; runDeps="$( scanelf --needed --nobanner --format '%n#p' --recursive "$TOMCAT_NATIVE_LIBDIR" | tr ',' '\n' | sort -u | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' 	)"; \
    apk add --virtual .tomcat-native-rundeps $runDeps; 	apk del .fetch-deps .native-build-deps; \
    apk add --no-cache bash; \
    find ./bin/ -name '*.sh' -exec sed -ri 's|^#!/bin/sh$|#!/usr/bin/env bash|' '{}' +; chmod -R +rX .; chmod 777 logs work

RUN set -e && \
    nativeLines="$(catalina.sh configtest 2>&1)" && \
    nativeLines="$(echo "$nativeLines" | grep 'Apache Tomcat Native')" && \
    nativeLines="$(echo "$nativeLines" | sort -u)" && \
    if ! echo "$nativeLines" | grep 'INFO: Loaded Apache Tomcat Native library' >&2; \
        then echo >&2 "$nativeLines"; \
        exit 1; \
    fi

EXPOSE 8080

CMD ["catalina.sh", "run"]