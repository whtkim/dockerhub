FROM ubuntu:22.04

ARG go_version="1.21.6"
ARG protoc_verstion="25.2"

ENV GO111MODULE="on"
ENV GOPROXY="https://goproxy.cn"
ENV GOPATH="/root/go"
ENV PATH=$PATH:$GOPATH/bin:/usr/local/go/bin:/usr/local/protoc/bin

RUN apt-get update -y \
 && apt-get install -y gcc g++ make curl zip net-tools telnet vim openssl ssh git inetutils-ping \
 && curl -L -o /tmp/go.tar.gz https://go.dev/dl/go${go_version}.linux-amd64.tar.gz \
 && tar -xvf /tmp/go.tar.gz -C /usr/local \
 && apt-get clean -y && rm -rf /tmp/* /var/tmp/* /var/cache/apt/*

RUN curl -L -o /tmp/protoc.zip https://github.com/protocolbuffers/protobuf/releases/download/v${protoc_verstion}/protoc-${protoc_verstion}-linux-x86_64.zip \
 && mkdir -p /usr/local/protoc && unzip /tmp/protoc.zip -d /usr/local/protoc \
 && go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28 \
 && go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2 \
 && go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest \
 && go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2@latest \
 && go install github.com/envoyproxy/protoc-gen-validate@v1.0.2 \
 && go install entgo.io/ent/cmd/ent@latest \
 && rm -rf /tmp/*