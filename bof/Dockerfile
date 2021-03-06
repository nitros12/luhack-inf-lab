# bof entrypoint

FROM ubuntu:latest AS build-vulnerable

RUN apt-get update && apt-get install -y \
    musl-tools \
    musl-dev

WORKDIR /opt/vulnerable/
COPY vulnerable.c .
RUN musl-gcc -static -no-pie -z execstack -fno-stack-protector -O0 vulnerable.c -o vulnerable_program

FROM clux/muslrust:latest AS build-rust

WORKDIR /opt/server/

COPY server/Cargo.toml .
COPY server/Cargo.lock .
RUN mkdir .cargo
RUN cargo vendor > .cargo/config

COPY server/src src
COPY --from=build-vulnerable /opt/vulnerable/vulnerable_program src/vulnerable_program

RUN cargo build --release
RUN cargo install --path . --verbose

FROM alpine:latest

ARG FLAG

RUN apk add util-linux

COPY --from=build-vulnerable /opt/vulnerable/vulnerable_program /usr/local/bin/vulnerable_program
COPY --from=build-rust /root/.cargo/bin/server /usr/local/bin/server

RUN adduser -g "" smooth -D -s /bin/sh
RUN su - smooth -c "echo $FLAG > /home/smooth/flag.txt"

CMD ["su", "-", "smooth", "-c", "server"]
