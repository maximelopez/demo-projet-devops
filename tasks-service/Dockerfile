# Stage : base (installe les dépendances communes)
FROM node:24-alpine AS base
WORKDIR /app
COPY package.json ./

# Stage : build (compile)
FROM base AS builder
RUN npm install
COPY . .
RUN npm run build

# Stage : development (pour dev, live-reload)
FROM base AS dev
COPY . .
CMD ["npm", "run", "dev"]

# Stage : production (image minimale et reproductible)
FROM node:24-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./
RUN npm install --production
EXPOSE 3001
CMD ["node", "dist/server.js"]
