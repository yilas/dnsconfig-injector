FROM golang:latest AS builder
COPY . $GOPATH/src/mypackage/myapp/
WORKDIR $GOPATH/src/mypackage/myapp/
RUN go mod init
RUN go mod tidy
#RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix nocgo -o /mutating-dns-webhook-server .
#RUN GOOS=linux go build -a nocgo -o /mutating-dns-webhook-server .
RUN CGO_ENABLED=0 GOOS=linux go build -o /mutating-dns-webhook-server .

FROM alpine:latest
#FROM debian:latest
COPY --from=builder /mutating-dns-webhook-server mutating-dns-webhook-server
#RUN mkdir /apps
#WORKDIR /apps
#COPY mutating-dns-webhook-server .
#RUN chmod 777 /apps/mutating-dns-webhook-server
ENTRYPOINT ["./mutating-dns-webhook-server"]
#ENTRYPOINT ["/lib/ld-musl-x86_64.so.1  /apps/mutating-dns-webhook-server"]
#ENTRYPOINT ["/apps/mutating-dns-webhook-server"]
