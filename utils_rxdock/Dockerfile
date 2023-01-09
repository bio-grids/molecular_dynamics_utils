FROM ubuntu:lunar-20221216

LABEL maintainer="DHN Chandan <biogrids.io@gmail.com>"
LABEL version="2023.1.1"
LABEL github="https://github.com/bio-grids/molecular_dynamics_utils"

# Update os
RUN apt-get update -y

# Install build dependencies
ARG build_deps="git meson build-essential cmake pkg-config ca-certificates"
RUN apt-get install --yes --no-install-recommends --quiet $build_deps

# Install app dependencies
ARG app_deps="libfmt-dev libeigen3-dev libpcg-cpp-dev googletest"
RUN apt-get install --yes --no-install-recommends --quiet $app_deps

# install rxdock
WORKDIR /
RUN git clone https://gitlab.com/rxdock/rxdock.git
WORKDIR /rxdock
RUN meson -Dbuildtype=release -Dtests=true builddir
RUN ninja -C builddir
# RUN ninja -C builddir test

ENV RBT_ROOT=/rxdock
ENV PATH=$RBT_ROOT/bin:$PATH
ENV PERL5LIB=$RBT_ROOT/lib:$PERL5LIB
ENV PATH=$RBT_ROOT/builddir:$PATH
ENV LD_LIBRARY_PATH=$RBT_ROOT/builddir:$LD_LIBRARY_PATH

# clean os
RUN rm -rf /tmp/*
RUN rm -rf /var/lib/apt/lists/*
RUN apt-get remove -y $build_deps
RUN apt-get autoremove -y
RUN apt-get clean -y

# make data work directory
RUN cd / && mkdir -p data
WORKDIR /data
