FROM golang:alpine as builder

COPY server-mock/go /go/src

WORKDIR /go/src

RUN go mod init h2server && go get golang.org/x/net/http2 && go build h2server.go

FROM alpine
COPY --from=builder /go/src/h2server /bin

CMD ["/bin/h2server"]