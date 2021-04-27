ARG GO_VER=1.16-alpine
ARG DOCKER_VER=20.10

FROM golang:${GO_VER} as go

RUN apk add \
      git \
      make

FROM go as nv2

RUN git clone https://github.com/notaryproject/nv2.git /nv2 \
 && cd /nv2 \
 && make
# && go build -o /usr/local/bin/nv2 ./cmd/nv2

FROM go as docker-generate

RUN git clone https://github.com/shizhMSFT/docker-generate.git /docker-generate \
 && cd /docker-generate \
 && go build -o /usr/local/bin/docker-generate ./cmd/docker-generate

FROM go as oras

RUN git clone https://github.com/deislabs/oras.git /oras \
 && cd /oras \
 && git checkout prototype-2 \
 && make build \
 && cp ./bin/linux/amd64/oras /usr/local/bin/oras

FROM docker:${DOCKER_VER} as docker

FROM alpine:latest

# COPY --from=nv2 /usr/local/bin/nv2 /usr/local/bin/nv2
COPY --from=nv2 /nv2/bin/ /usr/local/bin/
COPY --from=docker-generate /usr/local/bin/docker-generate /usr/local/bin/docker-generate
COPY --from=oras /usr/local/bin/oras /usr/local/bin/oras
COPY --from=docker /usr/local/bin/docker /usr/local/bin/docker
RUN apk add \
      bash \
      curl \
      git \
      jq \
      make \
      openssl

ENV HOME=/root
RUN mkdir -p /usr/libexec/docker/cli-plugins/ \
 && ln -s /usr/local/bin/docker-generate /usr/local/bin/docker-nv2 /usr/libexec/docker/cli-plugins/

WORKDIR /root
