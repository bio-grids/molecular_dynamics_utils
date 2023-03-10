ARG UBUNTU_BUILD_VER=22.04
ARG CUDA_VER=12.0.0

FROM nvidia/cuda:${CUDA_VER}-devel-ubuntu${UBUNTU_BUILD_VER} AS openmpi

ARG FFTW_VER=3.3.10
ARG GROMACS_VER=2022.4
ARG CMAKE_VER=3.25.1
ARG GCC_VER=11
ARG CUDA_VER=12.0.0
ARG OPENMPI_VER=4.1.4
ARG DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

# update os
RUN apt-get update -y

# install build dependencies
ARG build_deps="build-essential ca-certificates \
    g++-$GCC_VER gcc-$GCC_VER file make wget gpg-agent \
    bzip2 hwloc libnuma-dev openssh-client perl tar"
RUN apt-get install --yes --no-install-recommends --quiet $build_deps

# update gcc version
RUN update-alternatives --install /usr/bin/g++ g++ $(which g++-${GCC_VER}) 30 && \
    update-alternatives --install /usr/bin/gcc gcc $(which gcc-${GCC_VER}) 30 && \
    update-alternatives --install /usr/bin/gcov gcov $(which gcov-${GCC_VER}) 30

# install openmpi
WORKDIR /tmp
RUN wget https://www.open-mpi.org/software/ompi/v4.1/downloads/openmpi-${OPENMPI_VER}.tar.bz2
RUN tar -x -f openmpi-${OPENMPI_VER}.tar.bz2 -C . -j
WORKDIR /tmp/openmpi-${OPENMPI_VER}
RUN CC=gcc CXX=g++ ./configure --prefix=/usr/local/openmpi --disable-getpwuid --enable-orterun-prefix-by-default --with-cuda --without-verbs
RUN make -j$(nproc)
RUN make -j$(nproc) install

# set env
ENV LD_LIBRARY_PATH=/usr/local/openmpi/lib:$LD_LIBRARY_PATH
ENV PATH=/usr/local/openmpi/bin:$PATH

# install cmake
WORKDIR /tmp
RUN wget https://github.com/Kitware/CMake/releases/download/v${CMAKE_VER}/cmake-${CMAKE_VER}-linux-x86_64.sh
RUN /bin/sh cmake-${CMAKE_VER}-linux-x86_64.sh --prefix=/usr/local --skip-license

# set env
ENV PATH=/usr/local/bin:$PATH


FROM nvidia/cuda:${CUDA_VER}-devel-ubuntu${UBUNTU_BUILD_VER} AS gromacs

ARG FFTW_VER=3.3.10
ARG GROMACS_VER=2022.4
ARG CMAKE_VER=3.25.1
ARG GCC_VER=11
ARG CUDA_VER=12.0.0
ARG OPENMPI_VER=4.1.4
ARG DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

# update os
RUN apt-get update -y

# install build dependencies
ARG build_deps="build-essential ca-certificates \
    g++-$GCC_VER gcc-$GCC_VER file make wget gpg-agent \
    libblas-dev liblapack-dev ninja-build perl hwloc openssh-client"
RUN apt-get install --yes --no-install-recommends --quiet $build_deps

# update gcc version
RUN update-alternatives --install /usr/bin/g++ g++ $(which g++-${GCC_VER}) 30 && \
    update-alternatives --install /usr/bin/gcc gcc $(which gcc-${GCC_VER}) 30 && \
    update-alternatives --install /usr/bin/gcov gcov $(which gcov-${GCC_VER}) 30

# install cmake
WORKDIR /tmp
RUN wget https://github.com/Kitware/CMake/releases/download/v${CMAKE_VER}/cmake-${CMAKE_VER}-linux-x86_64.sh
RUN /bin/sh cmake-${CMAKE_VER}-linux-x86_64.sh --prefix=/usr/local --skip-license

# set env
ENV PATH=/usr/local/bin:$PATH

# copy from firesimulations fftw image
COPY --from=firesimulations/fftw:3.3.10 /usr/local/lib /usr/local/fftw/lib/
COPY --from=firesimulations/fftw:3.3.10 /usr/local/include /usr/local/fftw/include/

# set env
ENV CMAKE_PREFIX_PATH=/usr/local/fftw:$CMAKE_PREFIX_PATH

# copy openmpi
COPY --from=openmpi /usr/local/openmpi /usr/local/openmpi

# set env
ENV LD_LIBRARY_PATH=/usr/local/openmpi/lib:$LD_LIBRARY_PATH
ENV PATH=/usr/local/openmpi/bin:$PATH

LABEL gromacs.version=${GROMACS_VER}

# install gromacs
WORKDIR /tmp
RUN wget ftp://ftp.gromacs.org/pub/gromacs/gromacs-${GROMACS_VER}.tar.gz
RUN tar -x -f gromacs-${GROMACS_VER}.tar.gz -C . -z
RUN mkdir -p gromacs-${GROMACS_VER}/build
WORKDIR /tmp/gromacs-${GROMACS_VER}/build
RUN cmake .. \
    -D CMAKE_INSTALL_PREFIX=/usr/local/gromacs \
    -D CMAKE_BUILD_TYPE=Release \
    -D CMAKE_INSTALL_BINDIR=bin \
    -D CMAKE_INSTALL_LIBDIR=lib \
    -D CMAKE_C_COMPILER=mpicc \
    -D CMAKE_CXX_COMPILER=mpicxx \
    -D GMX_OPENMP=ON \
    -D GMX_MPI=ON \
    -D GMX_GPU=CUDA \
    -D GMX_USE_RDTSCP=OFF \
    -D GMX_DOUBLE=OFF \
    -D GMX_FFT_LIBRARY=fftw3 \
    -D GMX_EXTERNAL_BLAS=OFF \
    -D GMX_EXTERNAL_LAPACK=OFF \
    -D BUILD_SHARED_LIBS=ON \
    -D GMX_PREFER_STATIC_LIBS=ON \
    -D REGRESSIONTEST_DOWNLOAD=OFF \
    -D GMX_DEFAULT_SUFFIX=OFF \
    -D GMX_BINARY_SUFFIX=_mpi \
    -D GMX_LIBS_SUFFIX=_mpi \
    -G Ninja \
    -D MPIEXEC_PREFLAGS='--allow-run-as-root;--oversubscribe'
RUN cmake --build . --target all -- -j$(nproc)
RUN cmake --build . --target install -- -j$(nproc)
RUN rm -rf /tmp/gromacs-${GROMACS_VER} /tmp/gromacs-${GROMACS_VER}.tar.gz


FROM nvidia/cuda:${CUDA_VER}-runtime-ubuntu${UBUNTU_BUILD_VER}

LABEL maintainer="DHN Chandan <biogrids.io@gmail.com>"
LABEL version="2022.4"
LABEL github="https://github.com/bio-grids/molecular_dynamics_utils"

ARG FFTW_VER=3.3.10
ARG GROMACS_VER=2022.4
ARG CMAKE_VER=3.25.1
ARG GCC_VER=11
ARG CUDA_VER=12.0.0
ARG OPENMPI_VER=4.1.4
ARG DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

# update os
RUN apt-get update -y

# install app dependencies
ARG app_deps="python3 libgomp1 liblapack3 openmpi-bin openmpi-common openssh-client"
RUN apt-get install --yes --no-install-recommends --quiet $app_deps

# copy from firesimulations fftw image
COPY --from=firesimulations/fftw:3.3.10 /usr/local/lib /usr/local/fftw/lib/
COPY --from=firesimulations/fftw:3.3.10 /usr/local/include /usr/local/fftw/include/

# set env
ENV LD_LIBRARY_PATH=/usr/local/fftw/lib:$LD_LIBRARY_PATH

# copy compiled dependencies
COPY --from=openmpi /usr/local/openmpi /usr/local/openmpi

# set env
ENV LD_LIBRARY_PATH=/usr/local/openmpi/lib:$LD_LIBRARY_PATH
ENV PATH=/usr/local/openmpi/bin:$PATH

# copy gromacs
COPY --from=gromacs /usr/local/gromacs /usr/local/gromacs/

# set env
ENV PATH=$PATH:/usr/local/gromacs/bin

# "docker run --gpu 1" will bind /usr/lib/x86_64-linux-gnu/libcuda.so.1 found by
# x86_64-linux-gnu.conf, however for non-CUDA execution we'll put this
# as a fallback to avoid gmx warning messages about missing libcuda.so.1
RUN echo /usr/local/cuda/compat > /etc/ld.so.conf.d/zz-cuda-compat.conf && ldconfig

# clean os
RUN rm -rf /tmp/*
RUN rm -rf /var/lib/apt/lists/*
RUN apt-get autoremove -y
RUN apt-get clean -y

# make data work directory
RUN cd / && mkdir -p data
WORKDIR /data