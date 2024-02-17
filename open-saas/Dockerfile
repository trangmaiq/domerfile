FROM node AS base
RUN apk --no-cache -U upgrade # To ensure any potential security patches are applied.


FROM base AS server-builder
RUN apk add --no-cache build-base libtool autoconf automake
WORKDIR /app
COPY server/ ./server/
# Install npm packages, resulting in node_modules/.
RUN cd server && npm install
COPY db/schema.prisma ./db/
RUN cd server && PRISMA_CLIENT_OUTPUT_DIR=../server/node_modules/.prisma/client/ npx prisma generate --schema='../db/schema.prisma'
# Building the server should come after Prisma generation.
RUN cd server && npm run build


# TODO: Use pm2?
# TODO: Use non-root user (node).
FROM base AS server-production
# In case they want to use python3 in their app.
RUN apk add --no-cache python3
ENV NODE_ENV production
WORKDIR /app
COPY --from=server-builder /app/server/node_modules ./server/node_modules
COPY --from=server-builder /app/server/dist ./server/dist
COPY --from=server-builder /app/server/package*.json ./server/
COPY --from=server-builder /app/server/scripts ./server/scripts
COPY db/ ./db/
EXPOSE 3000
ENV PORT 3000
WORKDIR /app/server
ENTRYPOINT ["npm", "run", "start-production"]

# Any user-defined Dockerfile contents will be appended below.
