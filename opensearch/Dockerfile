# Use official OpenSearch image as base
FROM opensearchproject/opensearch:2.12.0

# Set environment variables for data directory
ENV OPENSEARCH_PATH_CONF=/usr/share/opensearch/config
ENV OPENSEARCH_HOME=/usr/share/opensearch
ENV path.data=/data

# Accept build arguments for credentials
ARG OPENSEARCH_U
ARG OPENSEARCH_P

# Set environment variables from build arguments
ENV OPENSEARCH_USERNAME=${OPENSEARCH_U}
ENV OPENSEARCH_PASSWORD=${OPENSEARCH_P}

# Install necessary tools and create data directory
RUN mkdir -p /data && \
    chown -R opensearch:opensearch /data && \
    chmod 755 /data

# Copy custom configuration files
COPY --chown=opensearch:opensearch opensearch.yml ${OPENSEARCH_PATH_CONF}/opensearch.yml
COPY --chown=opensearch:opensearch opensearch.keystore ${OPENSEARCH_HOME}/config/opensearch.keystore

# Switch to root for installing security plugin and certificates
USER root

# Install security dependencies and generate certificates
RUN yum install -y openssl && \
    ${OPENSEARCH_HOME}/bin/opensearch-plugin install --batch repository-s3 && \
    echo -e "y\n" | ${OPENSEARCH_HOME}/bin/opensearch-security/tools/securityadmin.sh \
    -cd ${OPENSEARCH_HOME}/config/opensearch-security \
    -icl -nhnv \
    -cacert ${OPENSEARCH_HOME}/config/root-ca.pem \
    -cert ${OPENSEARCH_HOME}/config/admin.pem \
    -key ${OPENSEARCH_HOME}/config/admin-key.pem

# Switch back to opensearch user
USER opensearch

# Expose HTTPS port
EXPOSE 9200

# Volume for persistent data
# VOLUME ["/data"]

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f -k -u ${OPENSEARCH_USERNAME}:${OPENSEARCH_PASSWORD} https://localhost:9200 || exit 1

# Start OpenSearch when the container starts
CMD ["/usr/share/opensearch/bin/opensearch", "-Ecluster.name=opensearch-cluster"]
