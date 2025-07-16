# # Private variables, not passed in from outside, but helpful for this file
ARG ASDF_VERSION=v0.18.0
ARG NODE_VERSION=22.16.0
ARG BUILD_REPO_CHEMOTION=https://github.com/ComPlat/chemotion_ELN

FROM ptrxyz/chemotion:eln-2.0.1

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

ARG ASDF_VERSION
ARG NODE_VERSION

# ASDF
RUN --mount=type=cache,target=/cache,sharing=locked \
    curl -sSL -o /cache/asdf.tar.gz "https://github.com/asdf-vm/asdf/releases/download/${ASDF_VERSION}/asdf-${ASDF_VERSION}-linux-amd64.tar.gz" && \
    tar -xzf /cache/asdf.tar.gz --one-top-level=/asdf && \
    chmod +x /asdf/asdf

# Ruby gems
RUN MAKEFLAGS="-j$(nproc)" && export MAKEFLAGS && \
    bundle install --jobs="$(nproc)" --retry=3 && \
    bundle clean --force

# NodeJS and modules
ENV ASDF_NODEJS_VERSION=${NODE_VERSION}

RUN --mount=type=cache,target=/asdf/cache,sharing=locked \
    MAKEFLAGS="-j$(nproc)" && export MAKEFLAGS && \
    asdf plugin add nodejs && \
    asdf install nodejs "${NODE_VERSION}" && \
    npm install -g yarn

RUN echo -e "--modules-folder ${NODE_PATH}\n--ignore-engines" > /chemotion/app/.yarnrc && \
    MAKEFLAGS="-j$(nproc)" && export MAKEFLAGS && \
    yarn install --modules-folder ${NODE_PATH} --ignore-engines --ignore-scripts 2>&1 | grep -v ^warning

RUN bash package_postinstall.sh