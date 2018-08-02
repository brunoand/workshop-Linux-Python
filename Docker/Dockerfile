# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
# Modified by Christian Diener
# Changes distributed under the MIT License.
FROM debian:jessie

MAINTAINER Christian Diener <mail@cdiener.com>

USER root

# Install all OS dependencies for fully functional notebook server
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get install -yq --no-install-recommends \
    git \
    wget \
    build-essential \
    python-dev \
    ca-certificates \
    bzip2 \
    unzip \
    sudo \
    locales \
    libatlas3-base \
    libfreetype6-dev \
    && apt-get clean
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Install Tini
RUN wget --quiet https://github.com/krallin/tini/releases/download/v0.9.0/tini && \
    mv tini /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini

# Configure environment
ENV CONDA_DIR /opt/conda
ENV PATH $CONDA_DIR/bin:$PATH
ENV SHELL /bin/bash
ENV NB_USER aluno
ENV NB_UID 1000
ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# Create aluno user with UID=1000 and in the 'users' group
RUN useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    mkdir -p /opt/conda && \
    chown aluno /opt/conda

USER aluno

# Setup aluno home directory
RUN mkdir /home/$NB_USER/work && \
    mkdir /home/$NB_USER/.jupyter && \
    mkdir /home/$NB_USER/.local

# Install conda as aluno
RUN cd /tmp && \
    mkdir -p $CONDA_DIR && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    /bin/bash Miniconda3-latest-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-latest-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda install --yes conda

# Install Jupyter notebook as aluno
RUN conda install --yes --quiet \
    jupyter \
    terminado \
    numpy \
    matplotlib=1.4.3 \
    pandas \
    scipy \
    && conda clean -yt

RUN conda install -c conda-forge jupyterlab

RUN mkdir -p $HOME/.config/matplotlib \
    && echo "backend: agg" > /home/aluno/.config/matplotlib/matplotlibrc \
    && fc-cache -fv

USER root

RUN echo 'b'
RUN git clone https://github.com/brunoand/16MP.git && \
   mv 16MP/start-notebook.sh /usr/local/bin/ && \
   mv 16MP/jupyter_notebook_config.py /home/$NB_USER/.jupyter/

# Configure container startup as root
EXPOSE 8888
WORKDIR /home/$NB_USER/work
ENTRYPOINT ["tini", "--"]
CMD ["start-notebook.sh"]

# Add local files as late as possible to avoid cache busting
RUN chmod a+rx /usr/local/bin/start-notebook.sh
RUN chown -R $NB_USER:users /home/$NB_USER/.jupyter
RUN echo aluno:aluno | chpasswd

# Switch back to aluno to avoid accidental container runs as root
USER aluno
