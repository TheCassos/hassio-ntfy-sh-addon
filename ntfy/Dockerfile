ARG BUILD_FROM
FROM $BUILD_FROM

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install requirements
RUN apk add --no-cache \
    ca-certificates \
    curl \
    jq \
    tzdata

# Set working directory
WORKDIR /app

# Download and install ntfy binary
RUN NTFY_VERSION=$(curl -s https://api.github.com/repos/binwiederhier/ntfy/releases/latest | jq -r .tag_name) && \
    ARCH=$(uname -m) && \
    case ${ARCH} in \
        aarch64) NTFY_ARCH="arm64" ;; \
        x86_64) NTFY_ARCH="amd64" ;; \
        armv7l) NTFY_ARCH="armv7" ;; \
        armv6l) NTFY_ARCH="armv6" ;; \
        *) echo "Unsupported architecture: ${ARCH}" && exit 1 ;; \
    esac && \
    curl -fsSL "https://github.com/binwiederhier/ntfy/releases/download/${NTFY_VERSION}/ntfy_${NTFY_VERSION#v}_linux_${NTFY_ARCH}.tar.gz" | tar -xz && \
    chmod +x ntfy && \
    mv ntfy /usr/local/bin/

# Copy run script
COPY run.sh /
RUN chmod a+x /run.sh

# Create data directory
RUN mkdir -p /data

# Set up user and permissions
RUN addgroup -g 1000 ntfy && \
    adduser -D -s /bin/sh -u 1000 -G ntfy ntfy && \
    chown -R ntfy:ntfy /data

# Switch to ntfy user
USER ntfy

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:8080/v1/health || exit 1

# Labels
LABEL \
    io.hass.name="ntfy" \
    io.hass.description="Self-hosted notification service" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version="${BUILD_VERSION}" \
    maintainer="Home Assistant Community" \
    org.opencontainers.image.title="ntfy Home Assistant Add-on" \
    org.opencontainers.image.description="Self-hosted notification service for Home Assistant" \
    org.opencontainers.image.source="https://github.com/binwiederhier/ntfy" \
    org.opencontainers.image.licenses="Apache-2.0"

# Run
CMD [ "/run.sh" ]