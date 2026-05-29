# Apache Jena CLI tools (tdb2.xloader, tdb2.tdbloader, jena.textindexer, …) for
# batch / loader workloads. The official Fuseki image is server-only and ships
# none of these tools, so this image packages the apache-jena distribution at the
# same Jena version as netwerkdigitaalerfgoed/fuseki (both fed from pom.xml).
#
# The base is pinned to a GNU-coreutils Ubuntu LTS on purpose: tdb2.xloader drives
# the load through bash + GNU `sort` (it relies on --parallel / --compress-program)
# plus `jq`. Alpine busybox and Ubuntu 25.10+ `uutils-coreutils` both produce a
# broken sort that corrupts the load (node-table “Bad hex char”, sort RC 141), so
# do NOT switch the base to plain `eclipse-temurin:21-jre` — it now tracks uutils.
FROM eclipse-temurin:21-jre-noble

ARG JENA_VERSION=6.1.0

RUN apt-get update \
 && apt-get install -y --no-install-recommends bash curl unzip jq ca-certificates \
 && rm -rf /var/lib/apt/lists/* \
 && curl -fsSL -o /tmp/jena.tar.gz \
      "https://repo1.maven.org/maven2/org/apache/jena/apache-jena/${JENA_VERSION}/apache-jena-${JENA_VERSION}.tar.gz" \
 && tar xzf /tmp/jena.tar.gz -C /opt \
 && ln -s "/opt/apache-jena-${JENA_VERSION}" /opt/jena \
 && rm /tmp/jena.tar.gz

ENV JENA_HOME=/opt/jena
ENV PATH="/opt/jena/bin:${PATH}"
