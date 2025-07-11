# Multi-stage build for SignalOS Server
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Production stage
FROM node:20-alpine AS production

WORKDIR /app

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S signalos -u 1001

# Copy built application
COPY --from=builder --chown=signalos:nodejs /app/dist ./dist
COPY --from=builder --chown=signalos:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=signalos:nodejs /app/package*.json ./

# Switch to non-root user
USER signalos

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Start the application
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/server/index.js"]