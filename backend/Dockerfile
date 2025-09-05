# backend/Dockerfile - production image

FROM node:18-alpine AS deps
WORKDIR /app

# copy only package.json + package-lock.json first (for caching)
COPY package*.json ./
RUN npm install --only=production

FROM node:18-alpine AS runtime
WORKDIR /app

# copy node_modules from deps
COPY --from=deps /app/node_modules ./node_modules

# copy rest of backend source code
COPY . .

ENV NODE_ENV=production
EXPOSE 5000

CMD ["node", "server.js"]
