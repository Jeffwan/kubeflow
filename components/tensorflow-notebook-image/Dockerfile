ARG BASE_IMAGE=763104351884.dkr.ecr.us-east-1.amazonaws.com/tensorflow-training:1.15.2-cpu-py36-ubuntu18.04

FROM $BASE_IMAGE

ARG NB_USER=jovyan

# TODO: User should be refactored instead of hard coded jovyan
USER root

ENV DEBIAN_FRONTEND noninteractive

ENV NB_USER $NB_USER

ENV NB_UID 1000
ENV HOME /home/$NB_USER
ENV NB_PREFIX /


# Use bash instead of sh
SHELL ["/bin/bash", "-c"]

RUN apt-get update && apt-get install -yq --no-install-recommends \
  apt-transport-https \
  build-essential \
  bzip2 \
  ca-certificates \
  curl \
  g++ \
  git \
  gnupg \
  graphviz \
  locales \
  lsb-release \
  openssh-client \
  sudo \
  unzip \
  vim \
  wget \
  zip \
  emacs \
  python3-pip \
  python3-dev \
  python3-setuptools \
  && apt-get clean && \
  rm -rf /var/lib/apt/lists/*

# Install Nodejs for jupyterlab-manager
RUN curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
RUN apt-get update && apt-get install -yq --no-install-recommends \
  nodejs \
  && apt-get clean && \
  rm -rf /var/lib/apt/lists/*

# Install kubectl client
RUN echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    apt-get update && \
    apt-get install -y kubectl

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# Create NB_USER user with UID=1000 and in the 'users' group
# but allow for non-initial launches of the notebook to have
# $HOME provided by the contents of a PV
RUN useradd -M -s /bin/bash -N -u $NB_UID $NB_USER && \
    chown -R ${NB_USER}:users /usr/local/bin && \
    mkdir -p $HOME

# Install Tini - used as entrypoint for container
RUN cd /tmp && \
    wget --quiet https://github.com/krallin/tini/releases/download/v0.18.0/tini && \
    echo "12d20136605531b09a2c2dac02ccee85e1b874eb322ef6baf7561cd93f93c855 *tini" | sha256sum -c - && \
    mv tini /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini

# NOTE: Beyond this point be careful of breaking out
# or otherwise adding new layers with RUN, chown, etc.
# The image size can grow significantly.

# Add EKS Kubeflow Workshop
# TODO: Pin to tag or version in later version.
RUN git clone https://github.com/aws-samples/eks-kubeflow-workshop.git $HOME

# Install base python3 packages
RUN pip3 --no-cache-dir install \
    jupyter-console==6.0.0 \
    jupyterlab \
    xgboost \
    kubeflow-fairing==0.7.1

COPY --chown=jovyan:users requirements.txt /tmp

# Configure container startup
EXPOSE 8888
USER jovyan
ENTRYPOINT ["tini", "--"]
CMD ["sh","-c", "jupyter notebook --notebook-dir=/home/${NB_USER} --ip=0.0.0.0 --no-browser --allow-root --port=8888 --NotebookApp.token='' --NotebookApp.password='' --NotebookApp.allow_origin='*' --NotebookApp.base_url=${NB_PREFIX}"]
