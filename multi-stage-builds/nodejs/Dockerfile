FROM node:24.0.0-slim AS ci

USER node

RUN mkdir /src/node/app
WORKDIR /src/node/app

RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/src/node/.npm,uid=1000,gid=1000,id=npm-cache \
    npm ci

COPY --chown=node:node . .

CMD ["tail", "-f", "/dev/null"]

FROM node:24.0.0-slim AS builder

ENV NODE_ENV=production

USER node

RUN mkdir /src/node/app
WORKDIR /src/node/app

COPY --chown=node:node --from=ci /src/node/app .
RUN npm run build

RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    --mount=type=cache,target=/src/node/.npm,uid=1000,gid=1000,id=npm-cache \
    npm ci --only=production

FROM node:24.0.0-slim AS production

ENV NODE_ENV=production

USER node

RUN mkdir /src/node/app
WORKDIR /src/node/app

COPY --chown=node:node --from=builder /src/node/app/dist ./dist
COPY --chown=node:node --from=builder /src/node/app/node_modules ./node_modules
COPY --chown=node:node --from=builder /src/node/app/package.json ./

EXPOSE 3000

CMD ["node", "dist/main.js"]
