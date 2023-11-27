FROM quay.io/fedora/fedora:latest

RUN dnf update -y &&\
    dnf install -y podman buildah skopeo &&\
    dnf clean all
