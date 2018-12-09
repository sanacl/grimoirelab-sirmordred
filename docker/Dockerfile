# Copyright (C) 2016-2017 Bitergia
# GPLv3 License

FROM debian:stretch-slim
MAINTAINER Jesus M. Gonzalez-Barahona <jgb@bitergia.com>, Alvaro del Castillo <acs@bitergia.com>

ENV GRIMOIRELAB_RELEASE "0.2.1"
ENV DEPLOY_USER bitergia
ENV DEPLOY_USER_DIR /home/${DEPLOY_USER}
ENV CONF_DIR ${DEPLOY_USER_DIR}/conf

# Initial user
RUN useradd ${DEPLOY_USER} --create-home --shell /bin/bash

# install dependencies
RUN apt-get update && \
    apt-get -y install --no-install-recommends \
        bash locales \
        gcc \
        git git-core \
        python3 \
        python3-pip \
        python3-venv \
        python3-dev \
        python3-gdbm \
        mariadb-client \
        unzip curl wget sudo ssh \
        && \
    apt-get clean && \
    find /var/lib/apt/lists -type f -delete

RUN echo "${DEPLOY_USER} ALL=NOPASSWD: ALL" >> /etc/sudoers

RUN mkdir -p ${DEPLOY_USER_DIR}/logs
RUN chown -R ${DEPLOY_USER}:${DEPLOY_USER} ${DEPLOY_USER_DIR}/logs

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout ${DEPLOY_USER_DIR}/logs/all.log \
    && ln -sf /dev/stderr ${DEPLOY_USER_DIR}/logs/all.log

# copy a default version of two configuration files
ADD https://raw.githubusercontent.com/chaoss/grimoirelab-sirmordred/master/aliases.json ${DEPLOY_USER_DIR}/aliases.json
ADD https://raw.githubusercontent.com/chaoss/grimoirelab-sirmordred/master/menu.yaml ${DEPLOY_USER_DIR}/menu.yml

COPY stage ${DEPLOY_USER_DIR}/stage

RUN chmod 444 ${DEPLOY_USER_DIR}/aliases.json
RUN chmod 444 ${DEPLOY_USER_DIR}/menu.yml
RUN chmod 755 ${DEPLOY_USER_DIR}/stage

USER ${DEPLOY_USER}

# Do the deployment for GRIMOIRELAB_RELEASE
WORKDIR ${DEPLOY_USER_DIR}

RUN mkdir -p ${CONF_DIR} && \
    wget https://raw.githubusercontent.com/chaoss/grimoirelab/master/releases/${GRIMOIRELAB_RELEASE} -O ${CONF_DIR}/requirements.cfg && \
    echo ${GRIMOIRELAB_RELEASE} > ${DEPLOY_USER_DIR}/release && \
    git clone https://github.com/grimoirelab/grimoirelab.git --depth 1 && \
    echo "Installing and checking GrimoireLab Release"
RUN sudo grimoirelab/utils/build_grimoirelab --build --install --check --install_system --relfile ${CONF_DIR}/requirements.cfg -l debug

CMD ${DEPLOY_USER_DIR}/stage
