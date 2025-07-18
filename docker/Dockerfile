# Stage 1: Build dal tuo fork
FROM node:20-alpine AS builder

# Installa dipendenze per il build
RUN apk add --no-cache \
    python3 \
    py3-pip \
    make \
    g++ \
    build-base \
    cairo-dev \
    pango-dev \
    git \
    curl

# Skip Chrome per Puppeteer durante il build
ENV PUPPETEER_SKIP_DOWNLOAD=true

# Crea directory di lavoro
WORKDIR /app

# Clona il tuo fork dal branch specifico
# IMPORTANTE: Sostituisci con i tuoi dati
ARG GITHUB_USERNAME
ARG GITHUB_BRANCH=main
RUN git clone -b ${GITHUB_BRANCH} https://github.com/${GITHUB_USERNAME}/flowise.git .

# Installa dipendenze e costruisci
RUN npm install
RUN npm run build

# Stage 2: Runtime
FROM node:20-alpine

# Installa runtime dependencies (come nel tuo Dockerfile originale)
RUN apk add --no-cache \
    chromium \
    git \
    python3 \
    py3-pip \
    make \
    g++ \
    build-base \
    cairo-dev \
    pango-dev \
    curl

# Set Puppeteer executable path
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
ENV PUPPETEER_SKIP_DOWNLOAD=true

# Crea user non-root per sicurezza
RUN addgroup -g 1001 -S flowise && \
    adduser -u 1001 -S flowise -G flowise

# Directory di lavoro
WORKDIR /app

# Copia i file buildati dal builder stage
COPY --from=builder /app/packages ./packages
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json

# Crea directories necessarie con permessi corretti
RUN mkdir -p /app/packages/server/uploads /app/packages/server/logs /app/packages/server/.flowise && \
    chown -R flowise:flowise /app

# Cambia all'utente non-root
USER flowise

# Esponi la porta
EXPOSE 3000

# Environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Comando di avvio
CMD ["node", "packages/server/dist/index.js"]
