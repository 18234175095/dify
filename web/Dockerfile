# base image
FROM node:20.11.0-alpine AS base
LABEL maintainer="takatost@gmail.com"

RUN apk add --no-cache tzdata


# install packages
FROM base as packages

WORKDIR /app/web

COPY package.json .
COPY yarn.lock .

RUN yarn install --frozen-lockfile


# build resources
FROM base as builder
WORKDIR /app/web
COPY --from=packages /app/web/ .
COPY . .

RUN yarn build


# production stage
FROM base as production

ENV NODE_ENV production
ENV EDITION SELF_HOSTED
ENV DEPLOY_ENV PRODUCTION
ENV CONSOLE_API_URL http://127.0.0.1:5001
ENV APP_API_URL http://127.0.0.1:5001
ENV PORT 3000

# set timezone
ENV TZ UTC
RUN ln -s /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone

WORKDIR /app/web
COPY --from=builder /app/web/public ./public
COPY --from=builder /app/web/.next/standalone ./
COPY --from=builder /app/web/.next/static ./.next/static


COPY docker/entrypoint.sh ./entrypoint.sh

ARG COMMIT_SHA
ENV COMMIT_SHA ${COMMIT_SHA}

EXPOSE 3000
ENTRYPOINT ["/bin/sh", "./entrypoint.sh"]
