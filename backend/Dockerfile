# ---------- Stage 1: Dependencies ----------
FROM node:20-alpine AS deps

WORKDIR /app

# Copy only dependency files (better caching)
COPY package.json package-lock.json ./

# Install production deps only
RUN npm install


# ---------- Stage 2: Runtime ----------
FROM node:20-alpine

WORKDIR /app

# Set production mode explicitly
ENV NODE_ENV=production

# Copy node_modules from deps stage
COPY --from=deps /app/node_modules ./node_modules

# Copy application code
COPY . .

# Use non-root user (security best practice)
USER node

# Expose backend port
EXPOSE 5000

# Healthcheck (VERY useful in Docker/K8s/PM2/VPS)
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s \
  CMD wget -qO- http://localhost:5000/health || exit 1

# Start application
CMD ["node", "server.js"]
