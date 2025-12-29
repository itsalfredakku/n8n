ARG NODE_VERSION=22.21.1

# ==============================================================================
# STAGE 1: Builder for Base Dependencies
# ==============================================================================
FROM node:${NODE_VERSION}-alpine AS builder

# Skip MS fonts installation (SourceForge mirrors are unreliable)
# Install fontconfig only for any system fonts
RUN apk --no-cache add fontconfig && fc-cache -f

# Install essential OS dependencies including build tools for native modules
RUN echo "https://dl-cdn.alpinelinux.org/alpine/v3.22/main" >> /etc/apk/repositories && \
    echo "https://dl-cdn.alpinelinux.org/alpine/v3.22/community" >> /etc/apk/repositories && \
    apk update && \
    apk upgrade --no-cache && \
    apk add --no-cache libxml2 && \
    apk add --no-cache \
        git \
        openssh \
        openssl \
        graphicsmagick \
        tini \
        tzdata \
        ca-certificates \
        libc6-compat \
        jq \
        python3 \
        make \
        g++

# Install full-icu
RUN npm install -g full-icu@1.5.0

RUN rm -rf /tmp/* /root/.npm /root/.cache/node

# ==============================================================================
# STAGE 2: Final Base Runtime Image
# ==============================================================================
FROM node:${NODE_VERSION}-alpine

COPY --from=builder / /

RUN rm -rf /opt/yarn* && apk del apk-tools

WORKDIR /home/node
ENV NODE_ICU_DATA=/usr/local/lib/node_modules/full-icu
EXPOSE 5678/tcp
