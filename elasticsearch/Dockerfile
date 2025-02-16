################################################################################
# This Dockerfile was generated from the template at distribution/src/docker/Dockerfile
#
# Beginning of multi stage Dockerfile
################################################################################

################################################################################
# Build stage 0 `builder`:
# Extract elasticsearch artifact
# Install required plugins
# Set gid=0 and make group perms==owner perms
################################################################################

FROM centos:7 AS builder

ENV PATH /usr/share/elasticsearch/bin:$PATH
ENV JAVA_HOME /opt/jdk-15+36

RUN ARCH= && case $(arch) in \
    x86_64) ARCH='x64';; \
    aarch64) ARCH='aarch64';; \
    *) echo "unsupported architecture - $(arch)"; exit 1 ;; \
  esac && \
  for iter in {1..10}; do curl -L -s -S https://github.com/AdoptOpenJDK/openjdk15-binaries/releases/download/jdk-15%2B36_openj9-0.23.0-m2/OpenJDK15U-jdk_${ARCH}_linux_openj9_15_36_openj9-0.23.0-m2.tar.gz | tar -C /opt -zx && \
    exit_code=0 && break || exit_code=\$? && echo "download error: ${ARCH} retry $iter in 10s" && sleep 10; done; \
    (exit $exit_code)
 
# Replace OpenJDK's built-in CA certificate keystore with the one from the OS
# vendor. The latter is superior in several ways.
# REF: https://github.com/elastic/elasticsearch-docker/issues/171
RUN ln -sf /etc/pki/ca-trust/extracted/java/cacerts ${JAVA_HOME}/lib/security/cacerts

RUN yum install -y unzip which

RUN groupadd -g 1000 elasticsearch && \
    adduser -u 1000 -g 1000 -d /usr/share/elasticsearch elasticsearch

WORKDIR /usr/share/elasticsearch

RUN cd /opt && curl --retry 8 -s -L -O https://artifacts-no-kpi.elastic.co/downloads/elasticsearch/elasticsearch-6.8.20.tar.gz && cd -

RUN tar zxf /opt/elasticsearch-6.8.20.tar.gz --strip-components=1
RUN grep ES_DISTRIBUTION_TYPE=tar /usr/share/elasticsearch/bin/elasticsearch-env \
    && sed -ie 's/ES_DISTRIBUTION_TYPE=tar/ES_DISTRIBUTION_TYPE=docker/' /usr/share/elasticsearch/bin/elasticsearch-env
RUN mkdir -p config data logs
RUN chmod 0775 config data logs
COPY config/elasticsearch.yml config/log4j2.properties config/

################################################################################
# Build stage 1 (the actual elasticsearch image):
# Copy elasticsearch from stage 0
# Add entrypoint
################################################################################

FROM debian:stable-slim

ENV ELASTIC_CONTAINER true
ENV JAVA_HOME /opt/jdk-15+36

COPY --from=builder /opt/jdk-15+36 /opt/jdk-15+36

RUN for iter in {1..10}; \
    do \
		  apt update  -y && apt install -y  netcat \
		  && apt autoremove -y && apt clean && exit_code=0 \
		  && break || exit_code=$? \
		  && echo "apt error: retry $iter in 10s" \
		  && sleep 10; \
		done;\
		(exit $exit_code) \
		&& groupadd -g 1000 elasticsearch && useradd -u 1000 -g 1000 -G 0 -m -d /usr/share/elasticsearch elasticsearch \
		&& chmod 0775 /usr/share/elasticsearch && chgrp 0 /usr/share/elasticsearch

WORKDIR /usr/share/elasticsearch
COPY --from=builder --chown=1000:0 /usr/share/elasticsearch /usr/share/elasticsearch
ENV PATH /usr/share/elasticsearch/bin:$PATH

COPY --chown=1000:0 bin/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

# Openshift overrides USER and uses ones with randomly uid>1024 and gid=0
# Allow ENTRYPOINT (and ES) to run even with a different user
RUN chgrp 0 /usr/local/bin/docker-entrypoint.sh && \
    chmod g=u /etc/passwd && \
    chmod 0775 /usr/local/bin/docker-entrypoint.sh

EXPOSE 9200 9300

LABEL org.label-schema.build-date="2021-10-07T22:00:24.085009Z" \
  org.label-schema.license="Elastic-License" \
  org.label-schema.name="Elasticsearch" \
  org.label-schema.schema-version="1.0" \
  org.label-schema.url="https://www.elastic.co/products/elasticsearch" \
  org.label-schema.usage="https://www.elastic.co/guide/en/elasticsearch/reference/index.html" \
  org.label-schema.vcs-ref="c859302dcfa51ee10915348ce467453130ed8b97" \
  org.label-schema.vcs-url="https://github.com/elastic/elasticsearch" \
  org.label-schema.vendor="Elastic" \
  org.label-schema.version="6.8.20" \
  org.opencontainers.image.created="2021-10-07T22:00:24.085009Z" \
  org.opencontainers.image.documentation="https://www.elastic.co/guide/en/elasticsearch/reference/index.html" \
  org.opencontainers.image.licenses="Elastic-License" \
  org.opencontainers.image.revision="c859302dcfa51ee10915348ce467453130ed8b97" \
  org.opencontainers.image.source="https://github.com/elastic/elasticsearch" \
  org.opencontainers.image.title="Elasticsearch" \
  org.opencontainers.image.url="https://www.elastic.co/products/elasticsearch" \
  org.opencontainers.image.vendor="Elastic" \
  org.opencontainers.image.version="6.8.20"

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
# Dummy overridable parameter parsed by entrypoint
CMD ["eswrapper"]

################################################################################
# End of multi-stage Dockerfile
################################################################################
