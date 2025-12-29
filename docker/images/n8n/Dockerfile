ARG NODE_VERSION=22.21.1
ARG N8N_VERSION=snapshot

# ==============================================================================
# STAGE 1: System Dependencies & Base Setup
# ==============================================================================
FROM n8nio/base:${NODE_VERSION} AS system-deps

# ==============================================================================
# STAGE 2: Build stage (with build tools for native modules)
# ==============================================================================
FROM node:${NODE_VERSION}-alpine AS builder

# Install build dependencies for native modules like sqlite3
RUN apk add --no-cache python3 make g++ gcc

ARG N8N_VERSION
ENV NODE_ENV=production

WORKDIR /home/node

COPY ./compiled /usr/local/lib/node_modules/n8n
COPY docker/images/n8n/docker-entrypoint.sh /

# Build sqlite3 from source for musl/Alpine compatibility
# Use --nodedir to avoid downloading headers from unofficial-builds.nodejs.org
RUN cd /usr/local/lib/node_modules/n8n/node_modules/.pnpm/sqlite3@5.1.7/node_modules/sqlite3 && \
    npx --yes node-gyp rebuild --nodedir=/usr/local && \
    ln -sf /usr/local/lib/node_modules/n8n/bin/n8n /usr/local/bin/n8n && \
    mkdir -p /home/node/.n8n && \
    chown -R node:node /home/node

# ==============================================================================
# STAGE 3: Final Runtime Image (without build tools)
# ==============================================================================
FROM system-deps AS runtime

ARG N8N_VERSION
ARG N8N_RELEASE_TYPE=dev
ENV NODE_ENV=production
ENV N8N_RELEASE_TYPE=${N8N_RELEASE_TYPE}
ENV NODE_ICU_DATA=/usr/local/lib/node_modules/full-icu
ENV SHELL=/bin/sh

WORKDIR /home/node

# Copy built artifacts from builder stage
COPY --from=builder /usr/local/lib/node_modules/n8n /usr/local/lib/node_modules/n8n
COPY --from=builder /usr/local/lib/node_modules/npm /usr/local/lib/node_modules/npm
COPY --from=builder /usr/local/bin/npm /usr/local/bin/npm
COPY --from=builder /docker-entrypoint.sh /
COPY --from=builder /home/node/.n8n /home/node/.n8n

# Recreate symlinks (COPY doesn't preserve symlinks properly across stages)
RUN ln -sf /usr/local/lib/node_modules/n8n/bin/n8n /usr/local/bin/n8n && \
    chown -R node:node /home/node

EXPOSE 5678/tcp
USER node
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]

LABEL org.opencontainers.image.title="n8n" \
      org.opencontainers.image.description="Workflow Automation Tool" \
      org.opencontainers.image.source="https://github.com/n8n-io/n8n" \
      org.opencontainers.image.url="https://n8n.io" \
      org.opencontainers.image.version=${N8N_VERSION}
