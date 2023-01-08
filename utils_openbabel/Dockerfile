FROM python:slim-bullseye

ARG DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

# Update os
RUN apt-get update -y

# Install build dependencies
ARG build_deps="\
    build-essential \
    ca-certificates \
    cmake \
    libboost-filesystem-dev \
    libboost-program-options-dev \
    libboost-regex-dev \
    libboost-system-dev \
    libboost-thread-dev \
    libboost-serialization-dev \
    libboost-python-dev \
    libboost-test-dev \
    libboost-dev \
    libcairo2-dev \
    libeigen3-dev \
    libxml2-dev \
    rapidjson-dev \
    wget \
    swig \
    zlib1g-dev \
    libbz2-dev \
    lzma \
    git"
RUN apt-get update -y
RUN apt-get install --yes --no-install-recommends --quiet $build_deps

# Install app dependencies
ARG app_deps="libboost-iostreams-dev libxml2"
RUN apt-get install --yes --no-install-recommends --quiet $app_deps

# Arguments
ARG OPENBABEL_VERSION=3.1.1
ARG OPENBABEL_DIR=3-1-1
ARG OPENBABEL_HOME=/usr/local/openbabel/$OPENBABEL_DIR

# Download and unzip openbabel
RUN wget --quiet --no-hsts --output-document=- https://github.com/openbabel/openbabel/archive/openbabel-${OPENBABEL_VERSION//./-}.tar.gz | tar -zxvf - -C /tmp
RUN mkdir -p /tmp/openbabel-openbabel-${OPENBABEL_VERSION//./-}/build
WORKDIR /tmp/openbabel-openbabel-${OPENBABEL_DIR}/build

# Build and install openbabel
RUN cmake \
    -W no-dev \
    -D RUN_SWIG=ON \
    -D PYTHON_BINDINGS=ON \
    -D PYTHON_EXECUTABLE=/usr/local/bin/python3.11 \
    -D CMAKE_INSTALL_PREFIX=$OPENBABEL_HOME \
    -D CMAKE_BUILD_TYPE=Release \
    ..
RUN make -j $(nproc) && make install

# Set environment variables
ENV LD_LIBRARY_PATH=$OPENBABEL_HOME/lib:$LD_LIBRARY_PATH
ENV PATH=$OPENBABEL_HOME/bin:$PATH

# Install openbabel python libraries
RUN pip install numpy pycairo
RUN pip install --install-option=build_ext --install-option="-I$OPENBABEL_HOME/include/openbabel3" --install-option="-L$OPENBABEL_HOME/lib" openbabel

# clean os
RUN rm -rf /tmp/*
RUN rm -rf /var/lib/apt/lists/*
RUN apt-get remove -y $build_deps
RUN apt-get autoremove -y
RUN apt-get clean -y

# make data work directory
RUN cd / && mkdir -p data
WORKDIR /data