ARG IMAGE_VER=2020.06

FROM nvcr.io/hpc/autodock:${IMAGE_VER}}

ARG VERSION=2023.1.1

LABEL maintainer="DHN Chandan <biogrids.io@gmail.com>"
LABEL version=${VERSION}
LABEL github="https://github.com/bio-grids/molecular_dynamics_utils"

# make data work directory
RUN cd / && mkdir -p data
WORKDIR /data