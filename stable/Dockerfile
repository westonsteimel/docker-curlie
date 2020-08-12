ARG REVISION
ARG CREATED
ARG VERSION
ARG GIT_REF="${VERSION}"

FROM golang:alpine AS builder

ARG GIT_REF
ARG VERSION
ENV VERSION="${VERSION}"
ENV CGO_ENABLED=0
ENV GOOS=linux
ENV GO111MODULE=on

RUN	apk upgrade && apk --no-cache add \
	ca-certificates \
    git \
    make \
    && rm -rf /var/cache

RUN mkdir -p /go/src/amass \
    && git clone --depth 1 --branch "${GIT_REF}" https://github.com/rs/curlie.git /go/src/curlie \
	&& cd /go/src/curlie \
    && go install -ldflags '-s -w' ./... \
	&& cp -vr /go/bin/* /usr/local/bin/ \
	&& if [$TARGETPLATFORM = "linux/amd64"]; then apk add --no-cache upx && upx --lzma --best /usr/local/bin/curlie; fi \
    && echo "Build complete."

RUN addgroup curlie \
    && adduser -G curlie -s /bin/sh -D curlie

FROM westonsteimel/curl:stable as curl

FROM scratch

ARG VERSION
ARG SOURCE
ARG REVISION
ARG CREATED

COPY --from=builder /usr/local/bin/curlie /usr/local/bin/curlie
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/ssl/certs /etc/ssl/certs
COPY --from=curl /usr/local/bin/curl /usr/local/bin/curl

USER curlie

ENTRYPOINT [ "/usr/local/bin/curlie" ]

LABEL org.opencontainers.image.title="curlie" \
    org.opencontainers.image.description="curlie in Docker" \ 
    org.opencontainers.image.source="${SOURCE}" \
    org.opencontainers.image.revision="${REVISION}" \
    org.opencontainers.image.created="${CREATED}" \
    org.opencontainers.image.version="${VERSION}"
