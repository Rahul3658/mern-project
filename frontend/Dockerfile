# ---- Stage 1: Build ----
FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .
RUN npm run build


# ---- Stage 2: Serve ----
FROM nginx:alpine

# copy files
COPY --from=builder /app/dist /usr/share/nginx/html

RUN chown -R nginx:nginx /usr/share/nginx/html
# use non-root user
USER nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
