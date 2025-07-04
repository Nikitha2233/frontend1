# Base image
FROM node:20-alpine AS base

WORKDIR /app

# Install system dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat

# Copy only package files
COPY package.json ./

# Install dependencies (no lockfile fallback needed, always install)
RUN npm install

# Copy source and build app
FROM base AS builder
WORKDIR /app

# Copy node_modules from deps stage
COPY --from=deps /app/node_modules ./node_modules

# Copy rest of app
COPY . .

# Build the app (standalone output)
RUN npm run build

# Final runtime image
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1
ENV PORT 8080

# Create non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# Create necessary directories and set permissions
RUN mkdir -p .next/standalone .next/static public && \
    chown -R nextjs:nodejs /app

# Copy built output
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public

USER nextjs

EXPOSE 8080

# Start the Next.js server
CMD ["node", "server.js"]
