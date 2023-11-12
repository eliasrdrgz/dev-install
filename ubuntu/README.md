# Ubuntu install

## OS & packages Upgrades

    sudo apt update && sudo apt upgrade -y && sudo apt clean && sudo apt remove -y && sudo apt autoremove -y && sdk selfupdate && sdk update

## sudo

## ssh

    sudo apt install openssh-server
    sudo systemctl enable ssh --now

    cat key.pub > ~/.ssh/authorized_keys
    chmod 600 ~/.ssh/authorized_keys

## tools

### general

#### zsh

    sudo apt install zsh

#### terminator

    sudo apt install terminator

#### sdkman

    curl -s "https://get.sdkman.io" | bash

### java

#### jdk

Prerequisite: `sdkman`

    sdk i java 17.0.5-tem

#### maven

Prerequisite: `sdkman`

    sdk i maven 3.8.7

    sdk i mcs 0.3.1

### containers

#### podman

    sudo apt -y install podman

#### docker-ce

    sudo apt-get remove docker docker-engine docker.io containerd runc
    sudo apt-get update
    sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

    sudo groupadd docker
    sudo usermod -aG docker $USER
    newgrp docker

    docker run --rm hello-world

#### docker-compose

See `docker-ce`

#### containers tools

##### stern

    cd /tmp && \
    curl -LO https://github.com/stern/stern/releases/download/v$(curl -L -s https://api.github.com/repos/stern/stern/releases/latest | grep -Po '"tag_name": "v\K[0-9.]+')/stern_$(curl -L -s https://api.github.com/repos/stern/stern/releases/latest | grep -Po '"tag_name": "v\K[0-9.]+')_linux_$(dpkg --print-architecture).tar.gz
    tar xzf stern_$(curl -L -s https://api.github.com/repos/stern/stern/releases/latest | grep -Po '"tag_name": "v\K[0-9.]+')_linux_$(dpkg --print-architecture).tar.gz
    sudo install stern_$(curl -L -s https://api.github.com/repos/stern/stern/releases/latest | grep -Po '"tag_name": "v\K[0-9.]+')_linux_$(dpkg --print-architecture) /usr/local/bin/stern

##### dive

    cd /tmp && \
    curl -LO https://github.com/wagoodman/dive/releases/download/v$(curl -s "https://api.github.com/repos/wagoodman/dive/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')/dive_$(curl -s "https://api.github.com/repos/wagoodman/dive/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')_linux_amd64.tar.gz
    tar xzf dive_$(curl -s "https://api.github.com/repos/wagoodman/dive/releases/latest" | grep -Po '"tag_name": "v\K[0-9.]+')_linux_amd64.tar.gz
    sudo install -o root -g root -m 0755 dive /usr/local/bin/dive

### kubernetes

#### kubectl

    cd /tmp && curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/$(dpkg --print-architecture)/kubectl"

    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

#### minikube

    cd /tmp && curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-$(dpkg --print-architecture)
    sudo install minikube-linux-$(dpkg --print-architecture) /usr/local/bin/minikube
    # for microcks
    minikube -p microcks start --cpus='8' --memory='16384' --addons ingress,ingress-dns

#### helm

TBD

#### k9s

    cd /tmp && wget https://github.com/derailed/k9s/releases/download/v0.28.1/k9s_Linux_amd64.tar.gz && \
    tar xzf k9s_Linux_amd64.tar.gz && \
    sudo install k9s /usr/local/bin/k9s && \
    cd -

#### knative

Prerequisites: `minikube`

Requires an architecture detection to automate the appropiate release, see [here](https://knative.dev/docs/getting-started/quickstart-install/#install-the-knative-cli)

##### knative cli

    cd /tmp && \
    curl -LO https://github.com/knative/client/releases/download/knative-v1.8.1/kn-linux-$(dpkg --print-architecture)
    sudo install kn-linux-$(dpkg --print-architecture) /usr/local/bin/kn

#### knative quickstart

    curl -LO https://github.com/knative-sandbox/kn-plugin-quickstart/releases/download/knative-v1.8.1/kn-quickstart-linux-$(dpkg --print-architecture)
    sudo install kn-quickstart-linux-$(dpkg --print-architecture) /usr/local/bin/kn-quickstart

    kn quickstart minikube

    minikube profile list

##### knative functions

    curl -LO https://github.com/knative/func/releases/download/knative-v$(curl -L -s https://api.github.com/repos/knative/func/releases/latest | grep -Po '"tag_name": "v\K[0-9.]+')/func_linux_$(dpkg --print-architecture)
    sudo install -o root -g root -m 0755 func /usr/local/bin

#### microcks

##### operator

    kubectl create namespace microcks
    kubectl apply -f https://microcks.io/operator/operator-latest.yaml -n microcks

###### minimal

    curl https://microcks.io/operator/minikube-minimal.yaml -s | sed 's/KUBE_APPS_URL/'$(minikube ip)'.nip.io/g' | kubectl apply -n microcks -f -

###### full features

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
    kubectl patch -n ingress-nginx deployment/ingress-nginx-controller --type='json' \
    -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--enable-ssl-passthrough"}]'
    kubectl apply -f 'https://strimzi.io/install/latest?namespace=microcks' -n microcks
    curl https://microcks.io/operator/minikube-features.yaml -s | sed 's/KUBE_APPS_URL/'$(minikube -p microcks ip)'.nip.io/g' | kubectl apply -n microcks -f -

##### helm

    helm repo add microcks https://microcks.io/helm
    helm repo update
    kubectl create namespace microcks
    helm install microcks microcks/microcks —-version 1.7.0 --namespace microcks --set microcks.url=microcks.$(minikube ip).nip.io --set keycloak.url=keycloak.$(minikube ip).nip.io

##### helm - asynchronous

    helm repo add microcks https://microcks.io/helm
    helm repo add strimzi https://strimzi.io/charts/
    helm repo update
    kubectl create namespace microcks
    helm install strimzi strimzi/strimzi-kafka-operator --namespace microcks
    helm install microcks microcks/microcks --namespace=microcks \
    --set appName=microcks --set features.async.enabled=true \
    --set microcks.url=microcks.$(minikube ip -p microcks).nip.io \
    --set keycloak.url=keycloak.$(minikube ip -p microcks).nip.io \
    --set features.async.kafka.url=$(minikube ip -p microcks).nip.io

    minikube -p microcks stop && minikube delete -p microcks 

### IDE

#### codium

    snap install codium --classic

#### visual studio code

    sudo apt install vscode

#### Spring Tool Suite

    cd /tmp && \
    curl -LO https://download.springsource.com/release/STS4/4.17.1.RELEASE/dist/e4.26/spring-tool-suite-4-4.17.1.RELEASE-e4.26.0-linux.gtk.x86_64.tar.gz
    tar xzf spring-tool-suite-4-4.17.1.RELEASE-e4.26.0-linux.gtk.x86_64.tar.gz

    rm spring-tool-suite-4-4.17.1.RELEASE-e4.26.0-linux.gtk.x86_64.tar.gz

### Graphs

#### drawio

    cd /tmp && \
    curl -LO https://github.com/jgraph/drawio-desktop/releases/download/v20.7.4/drawio-$(dpkg --print-architecture)-20.7.4.deb \
    sudo dpkg -i drawio-$(dpkg --print-architecture)-20.7.4.deb
