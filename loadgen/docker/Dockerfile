FROM ghcr.io/jgomezselles/hermes:0.0.4 as hermes1

FROM quay.io/fedora/fedora:38 as build2

RUN dnf install -y nghttp2 go

RUN go install github.com/rakyll/hey@latest
ENV PATH="${PATH}:/root/go/bin"

FROM quay.io/fedora/fedora:38

COPY --from=hermes1 /lib64/ /lib64/
COPY --from=build2 /lib64/ /lib64/

COPY --from=hermes1 /usr/local/bin/hermes /usr/local/bin/hermes
COPY --from=build2 /root/go/bin/hey /usr/local/bin/hey
COPY --from=build2 /usr/bin/h2load /usr/local/bin/h2load
