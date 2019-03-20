
# this dockerfile builds the image used as a CLI
ARG AZ_VER

FROM microsoft/azure-cli:${AZ_VER}

ARG HELM_VERSION
ARG FLUXCTL_VER=1.11.0

RUN az aks install-cli

RUN wget https://storage.googleapis.com/kubernetes-helm/helm-${HELM_VERSION}-linux-amd64.tar.gz && \
    tar -zxvf helm-${HELM_VERSION}-linux-amd64.tar.gz && \
    mv linux-amd64/helm /usr/local/bin/helm && \
    rm helm-${HELM_VERSION}-linux-amd64.tar.gz

RUN wget https://github.com/weaveworks/flux/releases/download/${FLUXCTL_VER}/fluxctl_linux_amd64 -O fluxctl && \
    mv fluxctl /usr/local/bin/fluxctl && \
    chmod +x /usr/local/bin/fluxctl
