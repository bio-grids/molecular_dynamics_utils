ARG IMAGE_VER=lunar-20221216

FROM ubuntu:${IMAGE_VER}

ARG VERSION=2023.1.1

LABEL maintainer="DHN Chandan <biogrids.io@gmail.com>"
LABEL version=${VERSION}
LABEL github="https://github.com/bio-grids/molecular_dynamics_utils"

# Update os
RUN apt-get update -y

# Install build dependencies
ARG build_deps="curl ca-certificates"
RUN apt-get install --yes --no-install-recommends --quiet $build_deps

# Install app dependencies
ARG app_deps="libxml2 libsm6 libxext6 libxrender-dev libgomp1"
RUN apt-get install --yes --no-install-recommends --quiet $app_deps

# install ADFRsuite
WORKDIR /tmp
RUN curl -SL https://ccsb.scripps.edu/adfr/download/1038 | tar -zx
RUN cd ADFRsuite_x86_64Linux_1.0; ./install.sh -d /opt/adfr/ -c 1 -l

ENV PATH=$PATH:/opt/adfr/bin/

# clean os
RUN rm -rf /tmp/*
RUN rm -rf /var/lib/apt/lists/*
RUN apt-get remove -y $build_deps
RUN apt-get autoremove -y
RUN apt-get clean -y

# copy bin files and add to path
WORKDIR /ubin
COPY bin/ .
ENV PATH=$PATH:/ubin/

# make data work directory
RUN cd / && mkdir -p data
WORKDIR /data