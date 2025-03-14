# syntax=docker/dockerfile:1.3-labs

FROM quay.io/devfile/base-developer-image:ubi9-latest
LABEL maintainer="cantin-f"

ENV HOME=/home/tooling
USER 0

# $PROFILE_EXT contains all additions made to the bash environment
ENV PROFILE_EXT=/etc/profile.d/udi_environment.sh
RUN touch ${PROFILE_EXT} && chown 10001 ${PROFILE_EXT}

# brotli, compat-openssl11 are required to run che-code
RUN dnf update -y && \
    dnf install -y bash bash-completion iputils sshpass \
                   brotli compat-openssl11 && \
    dnf clean all

# system module for python build
RUN dnf -y install gcc openldap-devel

# Python
RUN dnf -y install python3.12 python3.12-devel python3-setuptools python3-pip nss_wrapper && \
    dnf -y clean all --enablerepo='*'
RUN cd /usr/bin \
    && if [ ! -L python ]; then ln -s python3 python; fi \
    && if [ ! -L pydoc ]; then ln -s pydoc3 pydoc; fi \
    && if [ ! -L python-config ]; then ln -s python3-config python-config; fi

## Jfrog cli
RUN <<EOF
set -euf -o pipefail

cd /usr/local/bin
curl -fL https://getcli.jfrog.io | sh
chmod +x jfrog
EOF

## docker
RUN <<EOF
set -euf -o pipefail

dnf install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce docker-ce-cli containerd.io
EOF

## docker compose
RUN <<EOF
set -euf -o pipefail
TEMP_DIR="$(mktemp -d)"
cd "${TEMP_DIR}"
COMPOSE_VERSION="1.29.2"
COMPOSE_ARCH="Linux-x86_64"
COMPOSE_BIN="docker-compose-${COMPOSE_ARCH}"
COMPOSE_BIN_URL="https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/${COMPOSE_BIN}"
curl -sSLO "${COMPOSE_BIN_URL}"
curl -sSLO "${COMPOSE_BIN_URL}.sha256"
sha256sum -c "${COMPOSE_BIN}.sha256" 2>&1 | grep OK
chmod +x "${COMPOSE_BIN}"
mv "${COMPOSE_BIN}" /usr/local/bin/docker-compose
cd -
rm -rf "${TEMP_DIR}"
EOF

## kubectl
RUN <<EOF
set -euf -o pipefail
KUBECTL_VERSION="v1.25.5"

curl -L https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl -o /usr/bin/kubectl
chmod +x /usr/bin/kubectl
EOF

## oc
ENV OC_VERSION=4.17
RUN curl -L https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz | tar -C /usr/local/bin -xz \
    && chmod +x /usr/local/bin/oc

## helm
RUN <<EOF
set -euf -o pipefail
TEMP_DIR="$(mktemp -d)"
cd "${TEMP_DIR}"
HELM_VERSION="3.14.3"
HELM_ARCH="linux-amd64"
HELM_TGZ="helm-v${HELM_VERSION}-${HELM_ARCH}.tar.gz"
HELM_TGZ_URL="https://get.helm.sh/${HELM_TGZ}"
curl -sSLO "${HELM_TGZ_URL}"
curl -sSLO "${HELM_TGZ_URL}.sha256sum"
sha256sum -c "${HELM_TGZ}.sha256sum" 2>&1 | grep OK
tar -zxvf "${HELM_TGZ}"
mv "${HELM_ARCH}"/helm /usr/local/bin/helm
cd -
rm -rf "${TEMP_DIR}"
EOF

## Ansible
RUN <<EOF
set -euf -o pipefail
pip install ansible ansible-core ansible-compat molecule molecule-plugins[docker] docker ansible-lint
pip install boto3
pip install pywinrm
pip install pytest-testinfra
pip install flake8
EOF

## rancher
RUN <<EOF
set -euf -o pipefail
RANCHER_VERSION="v2.7.0"
curl -o- -L https://github.com/rancher/cli/releases/download/${RANCHER_VERSION}/rancher-linux-amd64-${RANCHER_VERSION}.tar.gz | tar xvz -C /usr/local/bin --strip-components=2
chmod +x /usr/local/bin/rancher
EOF

## vault
RUN <<EOF
set -euf -o pipefail
VAULT_VERSION="1.9.2"
TEMP_DIR="$(mktemp -d)"
curl -L https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip -o ${TEMP_DIR}/vault.zip
cd ${TEMP_DIR}
unzip vault.zip
mv vault /usr/bin/vault
chmod +x /usr/bin/vault
EOF

# Configure container engine
COPY --chown=0:0 containers.conf /etc/containers/containers.conf

# Configure auto completion
RUN <<EOF
oc completion bash > /usr/share/bash-completion/completions/oc
kubectl completion bash > /usr/share/bash-completion/completions/kubectl
EOF

RUN \
    # Configure sudo
    echo "user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/user &&\

    # Configure divenv
    echo 'eval "$(direnv hook bash)"' >> /home/tooling/.bashrc &&\
    
    chown -R 10001 /home/tooling

USER 10001

ENV HOME=/home/user
