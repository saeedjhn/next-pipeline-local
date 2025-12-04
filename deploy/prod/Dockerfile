###############################################
# Development Stage (optional)
###############################################
FROM node:24.11.1 AS development

ENV DIR=/app \
    NODE_ENV=development

WORKDIR $DIR

# Copy only deps files for caching
COPY package*.json ./

# Install all deps (including devDependencies)
RUN npm ci

# Copy rest of the source code
COPY . .

# Run Next.js Dev
CMD ["npm", "run", "dev"]


###############################################
# Builder Stage (Production build)
###############################################
FROM node:24.11.1 AS builder

ENV DIR=/app \
    NODE_ENV=production

WORKDIR $DIR

# Copy dependencies files
COPY package*.json ./

# Install only production dependencies + required build deps
RUN npm ci

# Copy full project
COPY . .

# Build next.js → .next/
RUN npm run build


###############################################
# Production Stage
###############################################
FROM node:24-alpine3.21 AS production

ENV DIR=/app \
    NODE_ENV=production

WORKDIR $DIR

# Copy only required package files
COPY package*.json ./

# Install only production dependencies for runtime
RUN npm ci --omit=dev

# Copy production build from builder
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/node_modules ./node_modules

# Next.js needs these for runtime
#COPY --from=builder /app/next.config.js ./next.config.js
COPY --from=builder /app/.next/standalone ./

# Expose the Next.js port
EXPOSE 3000

# Run Next.js production server
CMD ["npm", "run", "start"]
