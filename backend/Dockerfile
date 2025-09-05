# backend/Dockerfile - production image
FROM node:18-alpine AS deps
WORKDIR /app
COPY backend/package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS runtime
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY backend/ .
ENV NODE_ENV=production
EXPOSE 5000
CMD ["node", "server.js"]
