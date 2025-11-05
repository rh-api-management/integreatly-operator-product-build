# Upstream reference Dockerfile for drift detection purposes only
# Source: ocm-sendgrid-service/Dockerfile

# Build the binary
FROM registry-proxy.engineering.redhat.com/rh-osbs/openshift-golang-builder:v1.20 AS builder
ARG TARGETOS
ARG TARGETARCH

WORKDIR /workspace

# Add Gitlab certs
COPY test/support/2022-IT-Root-CA.crt /etc/pki/ca-trust/source/anchors/2022-IT-Root-CA.crt
COPY test/support/jwt_ca.pem /etc/pki/ca-trust/source/anchors/jwt_ca.pem
COPY test/support/gitlab-ca.crt /etc/pki/ca-trust/source/anchors/gitlab-ca.crt
RUN update-ca-trust

# Copy the Go Modules manifests
COPY go.mod go.mod
COPY go.sum go.sum

# Copy the vendor directory
COPY vendor/ vendor/

# Copy the go source
COPY cmd/ cmd/
COPY pkg/ pkg/
COPY data/ data/

# Build
# the GOARCH has not a default value to allow the binary be built according to the host where the command
# was called. For example, if we call make docker-build in a local env which has the Apple Silicon M1 SO
# the docker BUILDPLATFORM arg will be linux/arm64 when for Apple x86 it will be linux/amd64. Therefore,
# by leaving it empty we can ensure that the container and binary shipped on it will have the same platform.
RUN CGO_ENABLED=0 GOOS=${TARGETOS:-linux} GOARCH=${TARGETARCH} go build -mod=vendor -a -o service ./cmd/ocm-sendgrid-service

# Runner
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.5-240

RUN \
    microdnf install -y \
    openssl \
    && \
    microdnf clean all

COPY --from=builder /workspace/service /usr/local/bin/ocm-sendgrid-service

EXPOSE 8000

ENTRYPOINT ["/usr/local/bin/ocm-sendgrid-service", "serve"]

LABEL name="ocm-sendgrid-service" \
      vendor="Red Hat" \
      summary="OCM Sendgrid Service" \
      description="OCM Sendgrid Service"


