
# production images
FROM node:14-alpine
ENV TZ utc

ARG RELEASE_VERSION=0.23.2-beta.4
RUN apk add --no-cache tini git
ENTRYPOINT ["/sbin/tini", "--", "subql-node"]

RUN npm i -g bifrost-subql-node@${RELEASE_VERSION}
WORKDIR /workdir

CMD ["-f","/app"]
