# ---- Stage 1: Build ----
FROM node:20-alpine AS builder

WORKDIR /app

# Cache-friendly dependency install
COPY package.json package-lock.json ./

RUN npm install

# Copy source
COPY . .

# Build production bundle
RUN npm run build


# ---- Stage 2: Serve ----
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
