# # Private variables, not passed in from outside, but helpful for this file
# ARG RUBY_VERSION=2.7.8
# ARG NODE_VERSION=22.16.0
# ARG BUNDLER_VERSION=2.4.22
# ARG ASDF_VERSION=v0.18.0
# ARG BUILD_REPO_CHEMOTION=https://github.com/ComPlat/chemotion_ELN

FROM ptrxyz/chemotion:eln-2.0.1

# ENV ASDF_DIR=/asdf \
#     ASDF_DATA_DIR=/asdf \
#     GEM_HOME=/cache/gems \
#     ASDF_NODEJS_VERSION=22.16.0 \
#     ASDF_RUBY_VERSION=2.7.8

SHELL ["/bin/bash", "-e", "-o", "pipefail", "-c", "--"]

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends --autoremove --fix-missing \
        git ca-certificates curl unzip nano \
        build-essential \
        zlib1g-dev libreadline-dev patchelf \
        cmake libpq-dev swig libmagickcore-dev \
        postgresql-client inkscape imagemagick librsvg2-bin locales ghostscript \
        vim iproute2 sudo make

ARG BUILD_BRANCH

RUN git remote add origin https://github.com/ComPlat/chemotion_ELN && \
    git fetch && \
    git branch --set-upstream-to=origin/$BUILD_BRANCH && \
    git checkout $BUILD_BRANCH

RUN curl -sSL -o /cache/asdf.tar.gz "https://github.com/asdf-vm/asdf/releases/download/v0.18.0/asdf-v0.18.0-linux-amd64.tar.gz" && \
    tar -xzf /cache/asdf.tar.gz --one-top-level=/asdf && \
    chmod +x /asdf/asdf

RUN asdf plugin remove nodejs && \
    asdf plugin add nodejs && \
    asdf install # nodejs 22.16.0 && \
    npm install -g yarn

RUN MAKEFLAGS="-j$(nproc)" && export MAKEFLAGS && \
    yarn install --modules-folder ${NODE_PATH} --ignore-scripts 2>&1 | grep -v ^warning

RUN MAKEFLAGS="-j$(nproc)" && export MAKEFLAGS && \
    bundle install --jobs="$(nproc)" --retry=3 && \
    bundle clean --force
