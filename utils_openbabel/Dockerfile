ARG IMAGE_VER=3.11-slim-bullseye

FROM python:${IMAGE_VER}

ARG VERSION=2023.1.1
ARG OPENBABEL_VER=3.1.1

LABEL maintainer="DHN Chandan <biogrids.io@gmail.com>"
LABEL version=${VERSION}
LABEL github="https://github.com/bio-grids/molecular_dynamics_utils"

ARG DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

# Update os
RUN apt-get update -y

# Install app dependencies
ARG app_deps="openbabel libopenbabel-dev python3-numpy python3-cairo"
RUN apt-get install --yes --no-install-recommends --quiet $app_deps

# Install build dependencies
ARG build_deps="swig gcc g++"
RUN apt-get install --yes --no-install-recommends --quiet $build_deps

# install openbabel python library
RUN pip install --install-option=build_ext \
    --install-option="-I/usr/include/openbabel3" \
    --install-option="-L/usr/lib/x86_64-linux-gnu/openbabel/${OPENBABEL_VER}" \
    openbabel

# for converting python script to binaries
RUN pip install pyinstaller

# clean os
RUN rm -rf /tmp/*
RUN rm -rf /var/lib/apt/lists/*
RUN apt-get remove -y $build_deps
RUN apt-get autoremove -y
RUN apt-get clean -y

# make data work directory
RUN cd / && mkdir -p data
WORKDIR /data