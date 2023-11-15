# syntax=docker/dockerfile:1

FROM node:20.5-alpine3.17 AS build-stage

WORKDIR /usr/app

COPY package.json .

RUN npm install --quiet

COPY ./ ./

RUN npm run build

FROM nginx:stable-alpine AS build-release-stage

WORKDIR /

COPY --from=build-stage /usr/app/build /usr/share/nginx/html
