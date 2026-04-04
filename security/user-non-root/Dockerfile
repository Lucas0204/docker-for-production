FROM node:24.0.0-slim AS ci

# command to create a user on debian with home folder
# RUN useradd -m node
# command to create a user on alpine with home folder
# RUN adduser -D node
# node image does not have to create a user, because the user "node" already exists in the base image

USER node

# here it is needed to create the folder before the "WORKDIR" instruction,
# because, if the folder does not exists, the "WORKDIR" instruction will create it with root permissions instead of using the user "node" permissions
RUN mkdir /usr/src/app
WORKDIR /usr/src/app

RUN --mount=type=bind,source=package.json,target=package.json \
  --mount=type=bind,source=package-lock.json,target=package-lock.json \
  --mount=type=cache,target=/usr/src/.npm,uid=1000,gid=1000,id=npm-cache \
  npm ci

# copy the source code with user "node" as owner
COPY --chown=node:node . .

CMD ["tail", "-f", "/dev/null"]

FROM node:24.0.0-slim AS builder

ENV NODE_ENV=production

USER node

RUN mkdir /usr/src/app
WORKDIR /usr/src/app

COPY --from=ci --chown=node:node /usr/src/app .
RUN npm run build

RUN --mount=type=cache,target=/usr/src/.npm,uid=1000,gid=1000,id=npm-cache \
  npm ci --only=production

FROM node:24.0.0-slim AS production

ENV NODE_ENV=production


# Secure definition to logs folder:

# 1.
# never do the folowing RUN instruction, because the folder /var/log is originally owned by root,
# and also the content that's been right there may expose sensitive data. It's mean that the user "node"
# cannot have access to the entire folder /var/log, because it may cause security issues
# RUN chown node:node /var/log

# 2. 
# the following instruction is better but not the ideal. It's limiting the permissions
# to a group that user "node" is part of, so the access is limited to root and members of the group.
# Altough, the members of the group may have access to the entire folder /var/log, which may cause security issues
# RUN groupadd logswriters && \
#   usermod -aG logswriters node && \
#   chown root:logswriters /var/log && \
#   chmod 775 /var/log

# 3.
# the secure way to do this is to create a specific folder to application logs,
# for it's owned by the user "node" only, and the entire /var/log folder is accessible only by the root user,
# that is, the user "node" can only access the specific folder inside /var/log and not the entire folder.
# It prevents security issues by limiting the access of the user of the container, obeying the rule of least privilege
RUN mkdir /var/log/app && \
  chown node:node /var/log/app

USER node

RUN mkdir /usr/src/app
WORKDIR /usr/src/app

COPY --chown=node:node --from=builder /usr/src/app/dist ./dist
COPY --chown=node:node --from=builder /usr/src/app/node_modules ./node_modules
COPY --chown=node:node --from=builder /usr/src/app/package.json ./

EXPOSE 3000

CMD ["node", "dist/main.js"]
