# excalidraw/excalidraw-room does publish an image, but its last build
# (December 2023) runs on node:12, end-of-life since April 2022, as root.
# Upstream's code has not changed since that build. This rebuilds the same
# commit on a supported Node runtime, distroless and non-root. No source of
# ours.

# renovate: datasource=git-refs depName=https://github.com/excalidraw/excalidraw-room currentValue=master
ARG COMMIT=03ff435860b508d7cd9e005cfc90f7977ae2a593

FROM node:24-slim AS build
ARG COMMIT
# BuildKit fetches the exact commit itself: no git in the image.
ADD https://github.com/excalidraw/excalidraw-room.git#${COMMIT} /src
WORKDIR /src
# yarn classic ships in the node image; upstream pins no packageManager, so
# corepack would resolve "latest" instead. Upstream lists every package,
# typescript and eslint included, under `dependencies`, so there is no lighter
# production install to make.
RUN yarn install --frozen-lockfile --non-interactive \
 && yarn build

FROM gcr.io/distroless/nodejs24-debian13:nonroot
# express.static("public") and dotenv resolve against the working directory.
WORKDIR /app
COPY --from=build /src/package.json ./
COPY --from=build /src/node_modules ./node_modules
COPY --from=build /src/dist ./dist
COPY --from=build /src/public ./public
# Upstream defaults to port 80 outside development, which needs root.
ENV NODE_ENV=production \
    PORT=8080
EXPOSE 8080
CMD ["dist/index.js"]
