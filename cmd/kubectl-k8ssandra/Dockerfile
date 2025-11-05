# Start from UBI minimal for yq and kubectl installation
FROM redhat/ubi9-minimal:latest AS ubi-builder

ARG YQ_VERSION
ARG KUBE_VERSION

# Install kubectl
RUN cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v${KUBE_VERSION:-1.33}/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v${KUBE_VERSION:-1.33}/rpm/repodata/repomd.xml.key
EOF

RUN microdnf update -y
RUN microdnf install --nodocs -y kubectl ca-certificates

WORKDIR /workspace

# Install yq
RUN curl -L "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION:-4.47.1}/yq_linux_amd64" -o yq

# Final image
FROM redhat/ubi9-micro:latest

ARG VERSION

WORKDIR /workspace

# Copy the pre-built Go binary from the build context
COPY kubectl-k8ssandra /workspace/kubectl-k8ssandra

# Copy yq and kubectl from the ubi-builder stage
COPY --from=ubi-builder /workspace/yq /usr/bin/yq
COPY --from=ubi-builder /usr/bin/kubectl /usr/bin/kubectl

# Copy CA certificates
COPY --from=ubi-builder /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem /etc/pki/ca-trust/extracted/pem/

COPY ./LICENSE /licenses/

USER 65532:65532

ENTRYPOINT ["/workspace/kubectl-k8ssandra"]
