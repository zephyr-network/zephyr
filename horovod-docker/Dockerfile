FROM ubuntu:16.04

RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        cmake \
        git \
        curl \
        vim \
        wget \
        ca-certificates \
        libjpeg-dev \
        libpng-dev 

# Install TensorFlow and Keras
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends curl && \
  curl -sL http://repo.radeon.com/rocm/apt/debian/rocm.gpg.key | apt-key add - && \
  sh -c 'echo deb [arch=amd64] http://repo.radeon.com/rocm/apt/debian/ xenial main > /etc/apt/sources.list.d/rocm.list' && \
  apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  sudo \
  libelf1 \
  libnuma-dev \
  build-essential \
  git \
  vim-nox \
  cmake-curses-gui \
  rocm-dev && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*

ENV PATH "${PATH}:/opt/rocm/bin"

ENV ROCM_PATH /opt/rocm
ENV DEBIAN_FRONTEND noninteractive

RUN apt update && apt install -y wget software-properties-common && \
    apt-get update && apt-get install -y \
  build-essential \
  clang-3.8 \
  clang-format-3.8 \
  clang-tidy-3.8 \
  cmake \
  cmake-qt-gui \
  ssh \
  curl \
  apt-utils \
  pkg-config \
  g++-multilib \
  git \
  libunwind-dev \
  libfftw3-dev \
  libelf-dev \
  libncurses5-dev \
  libpthread-stubs0-dev \
  vim \
  gfortran \
  libboost-program-options-dev \
  libssl-dev \
  libboost-dev \
  libboost-system-dev \
  libboost-filesystem-dev \
  rpm \
  wget && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* && \
  apt-get update && \
  apt-get install -y --allow-unauthenticated \
  rocm-dev rocm-libs \
  rocm-device-libs \
  hsa-ext-rocr-dev hsakmt-roct-dev hsa-rocr-dev \
  rocm-opencl rocm-opencl-dev \
  rocm-utils \
  rocm-profiler cxlactivitylogger \
  miopen-hip miopengemm && \
  apt-get update && apt-get install -y \
    python3-numpy \
    python3-dev \
    python3-wheel \
    python3-mock \
    python3-future \
    python3-pip \
    python3-yaml \
    python3-setuptools && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN cd ~/ && wget http://repo.radeon.com/rocm/misc/tensorflow/tensorflow-1.8.0-cp35-cp35m-manylinux1_x86_64.whl && \
    pip3 install tensorflow-1.8.0-cp35-cp35m-manylinux1_x86_64.whl && rm tensorflow-1.8.0-cp35-cp35m-manylinux1_x86_64.whl

RUN pip3 install keras h5py

RUN ln -s /usr/bin/pip3 /usr/bin/pip && \
    apt-get update && apt-get -y install cargo && \
    cargo install sccache

# Install Open MPI
RUN mkdir /tmp/openmpi && \
    cd /tmp/openmpi && \
    wget https://www.open-mpi.org/software/ompi/v3.0/downloads/openmpi-3.0.0.tar.gz && \
    tar zxf openmpi-3.0.0.tar.gz && \
    cd openmpi-3.0.0 && \
    ./configure --enable-orterun-prefix-by-default && \
    make -j $(nproc) all && \
    make install && \
    ldconfig && \
    rm -rf /tmp/openmpi

RUN HOROVOD_WITH_TENSORFLOW=1 pip3 install --no-cache-dir git+https://github.com/ROCmSoftwarePlatform/horovod.git

# Create a wrapper for OpenMPI to allow running as root by default
RUN mv /usr/local/bin/mpirun /usr/local/bin/mpirun.real && \
    echo '#!/bin/bash' > /usr/local/bin/mpirun && \
    echo 'mpirun.real --allow-run-as-root "$@"' >> /usr/local/bin/mpirun && \
    chmod a+x /usr/local/bin/mpirun

# Configure OpenMPI to run good defaults:
#   --bind-to none --map-by slot --mca btl_tcp_if_exclude lo,docker0
RUN echo "hwloc_base_binding_policy = none" >> /usr/local/etc/openmpi-mca-params.conf && \
    echo "rmaps_base_mapping_policy = slot" >> /usr/local/etc/openmpi-mca-params.conf && \
    echo "btl_tcp_if_exclude = lo,docker0" >> /usr/local/etc/openmpi-mca-params.conf

# Install OpenSSH for MPI to communicate between containers
RUN apt-get install -y --no-install-recommends openssh-client openssh-server && \
    mkdir -p /var/run/sshd

# Allow OpenSSH to talk to containers without asking for confirmation
RUN cat /etc/ssh/ssh_config | grep -v StrictHostKeyChecking > /etc/ssh/ssh_config.new && \
    echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config.new && \
    mv /etc/ssh/ssh_config.new /etc/ssh/ssh_config

# Download examples
RUN apt-get install -y --no-install-recommends subversion && \
    svn checkout https://github.com/uber/horovod/trunk/examples && \
    rm -rf /examples/.svn

RUN mkdir -p /root/imagenet && \
    mkdir -p /root/imagenet/train && \
    mkdir -p /root/imagenet/validation

WORKDIR "/examples"
