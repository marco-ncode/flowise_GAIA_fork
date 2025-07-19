# Stage 1: Build Flowise from your fork
FROM node:20-alpine AS build
USER root

# Skip downloading Chrome for Puppeteer
ENV PUPPETEER_SKIP_DOWNLOAD=true

# Install build dependencies
RUN apk add --no-cache chromium git python3 py3-pip make g++ build-base cairo-dev pango-dev curl

# Set the NODE_OPTIONS environment variable to increase the Node.js heap size
ENV NODE_OPTIONS=--max_old_space_size=8192

# Arguments for your fork
ARG GITHUB_USERNAME
ARG GITHUB_BRANCH=main

# Temporary directory for the build
WORKDIR /build

# Clone and build your fork
RUN git clone -b ${GITHUB_BRANCH} https://github.com/${GITHUB_USERNAME}/flowise.git . && \
    # Check and use the appropriate package manager
    if [ -f "yarn.lock" ]; then \
        echo "Using Yarn" && \
        npm install -g yarn && \
        yarn install && \
        yarn build; \
    elif [ -f "pnpm-lock.yaml" ]; then \
        echo "Using PNPM" && \
        npm install -g pnpm@9 && \
        pnpm install && \
        pnpm build; \
    else \
        echo "Using NPM" && \
        npm install -g npm@latest && \
        npm ci --workspaces --include-workspace-root && \
        npm run build; \
    fi

# Stage 2: Runtime (like your original)
FROM node:20-alpine

# Install runtime dependencies
RUN apk add --no-cache chromium git python3 py3-pip make g++ build-base cairo-dev pango-dev curl

# Set the environment variable for Puppeteer to find Chromium
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
ENV PUPPETEER_SKIP_DOWNLOAD=true

# Create and set the working directory
WORKDIR /usr/src/app

# Copy the built project from the build stage
COPY --from=build /build /usr/src/app

# Create data directory
RUN mkdir -p /root/.flowise

# Adjust the entrypoint to run from the local node_modules
ENTRYPOINT ["node", "packages/server/dist/index.js"]
