FROM python:slim-bullseye

ARG DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

# Update os
RUN apt-get update -y

# Install app dependencies
ARG app_deps="openbabel"
RUN apt-get install --yes --no-install-recommends --quiet $app_deps

# make data work directory
RUN cd / && mkdir -p data
WORKDIR /data