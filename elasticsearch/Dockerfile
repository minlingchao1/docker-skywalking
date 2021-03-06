FROM openjdk:8-jre-alpine

LABEL maintainer="Badboy <287564883@@qq.com>"

ARG TZ="Asia/Shanghai"

ENV TZ ${TZ}
# https://artifacts.elastic.co/GPG-KEY-elasticsearch
ENV GPG_KEY 46095ACC8548582C1A2699A9D27D666CD88E42B4
ENV ELASTICSEARCH_VERSION 5.6.10
ENV ELASTICSEARCH_HOME /usr/local/elasticsearch
ENV ELASTICSEARCH_TARBALL https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-${ELASTICSEARCH_VERSION}.tar.gz
ENV ELASTICSEARCH_TARBALL_ASC https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-${ELASTICSEARCH_VERSION}.tar.gz.asc
ENV ELASTICSEARCH_TARBALL_SHA1 8d5c6a95ab7569eee42a8430778111c1944a6f4e

# COPY elasticsearch/sources.list /sources.list 
# RUN mv /etc/apt/sources.list /etc/apt/sources.list .bak
# RUN mv /sources.list /etc/apt/sources.list 
# ensure elasticsearch user exists

# 备份原始文件 
RUN cp /etc/apk/repositories /etc/apk/repositories.bak 
# 修改为国内镜像源 
RUN echo "http://mirrors.aliyun.com/alpine/v3.4/main/" > /etc/apk/repositories \
&& addgroup -S elasticsearch \
&& adduser -S -G elasticsearch elasticsearch \
&& apk upgrade --update \
    # grab su-exec for easy step-down from root
    # and bash for "bin/elasticsearch" among others
&& apk add 'su-exec>=0.2' bash tzdata coreutils \
&& apk add  ca-certificates  gnupg   openssl  tar \
&& wget -O elasticsearch.tar.gz "${ELASTICSEARCH_TARBALL}" \
&&  wget -O elasticsearch.tar.gz.asc "${ELASTICSEARCH_TARBALL_ASC}" \
#RUN  echo "${ELASTICSEARCH_TARBALL_SHA1}  elasticsearch.tar.gz" | sha1sum -c - 
&&  export GNUPGHOME="$(mktemp -d)" \
&&  gpg --keyserver pgp.mit.edu --recv-keys "${GPG_KEY}" \
&&  gpg --batch --verify elasticsearch.tar.gz.asc elasticsearch.tar.gz \
&&  tar -zxf elasticsearch.tar.gz \
&&  mv elasticsearch-${ELASTICSEARCH_VERSION} ${ELASTICSEARCH_HOME} \
&&  mkdir -p /data/elasticsearch /var/log/elasticsearch \
&& chown -R elasticsearch:elasticsearch ${ELASTICSEARCH_HOME} /data/elasticsearch /var/log/elasticsearch \
&& ln -sf ${ELASTICSEARCH_HOME}/bin/* /usr/local/bin \
&&  ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
&&  echo ${TZ} > /etc/timezone 
#RUN  apk del .fetch-deps 
RUN  rm -rf "${GNUPGHOME}" \
        elasticsearch.tar.gz.asc \
        elasticsearch.tar.gz \
        ${ELASTICSEARCH_HOME}/bin/*.exe \
        ${ELASTICSEARCH_HOME}/bin/*.bat \
    && rm -rf /var/cache/apk/*  \
    && export ES_JAVA_OPTS="-Xms256m -Xmx256m"

#COPY config ${ELASTICSEARCH_HOME}/config

COPY docker-entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
VOLUME /data/elasticsearch /var/log/elasticsearch 

EXPOSE 9200 9300

ENTRYPOINT ["/entrypoint.sh"]

CMD ["elasticsearch"]