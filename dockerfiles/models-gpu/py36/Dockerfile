ARG CUDA_VER=10.1

FROM nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04

LABEL maintainer="yashab@iguazio.com"
LABEL org="iguazio.com"

ARG OMPI=4.0.3
ARG REPO=mlrun

ARG MLRUN_TAG=development
ENV MLRUN_TAG=${MLRUN_TAG}

ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV PATH /opt/conda/bin:$PATH

# Set default shell to /bin/bash
SHELL ["/bin/bash", "-cu"]

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    build-essential \
    bzip2 \
    cmake \
    curl \
    git \
    ibverbs-providers \
    libboost-dev \
    libboost-filesystem-dev \
    libboost-system-dev \
    libc6 \
    libglib2.0-0 \
    libibverbs1\
    libjpeg-dev \
    libnvinfer-plugin6=6.0.1-1+cuda10.1 \
    libnvinfer6=6.0.1-1+cuda10.1 \
    libpng-dev \
    librdmacm1 \
    libreadline-dev \
    libreadline7 \
    libsm6 \
    libxext6 \
    libxrender1 \
    mercurial \
    openssh-client \
    openssh-server \
    subversion \
    wget && \
    apt install -y ca-certificates  && \
    update-ca-certificates --fresh && \
    apt clean && \
    apt autoremove && \
    rm -rf /var/lib/apt/lists/*

ENV SSL_CERT_DIR /etc/ssl/certs

RUN mkdir -p /var/run/sshd && \
    # Allow OpenSSH to talk to containers without asking for confirmation
    cat /etc/ssh/ssh_config | grep -v StrictHostKeyChecking > /etc/ssh/ssh_config.new && \
    echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config.new && \
    mv /etc/ssh/ssh_config.new /etc/ssh/ssh_config && \
    svn checkout https://github.com/horovod/horovod/trunk/examples && \
    rm -rf /examples/.svn

# Install Open MPI
RUN mkdir /tmp/openmpi && \
    cd /tmp/openmpi && \
    curl -o openmpi-${OMPI}.tar.gz https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-${OMPI}.tar.gz && \
    tar zxf openmpi-${OMPI}.tar.gz && \
    cd openmpi-${OMPI} && \
    ./configure --enable-orterun-prefix-by-default && \
    make -j`nproc` all && \
    make install && \
    ldconfig && \
    rm -rf /tmp/openmpi    

ENV OMPI_ALLOW_RUN_AS_ROOT=1
ENV OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1

RUN wget --quiet https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/installconda.sh && \
    /bin/bash ~/installconda.sh -b -p /opt/conda && \
    rm ~/installconda.sh && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc

RUN conda config --add channels bokeh && \
    conda config --add channels conda-forge && \
    conda config --add channels anaconda && \
    conda update -n base conda && \
    conda install -n base python=3.6 \
    bokeh \
    cmake \
    cudatoolkit=10.1 \
    cython \
    dask-kubernetes \
    dask-ml \
    dask-xgboost \
    dask[complete] \
    distributed \
    fsspec \
    matplotlib \
    numba \
    numpy \
    pandas \
    pyarrow \
    pytest \
    scikit-build \
    scikit-learn \
    scikit-optimize \
    scipy \
    seaborn \
    tensorflow-gpu \
    wrapt

RUN conda install -n base -c rapidsai -c nvidia \
    -c anaconda -c conda-forge -c defaults rapids=0.12

RUN conda install -n base -c pytorch pytorch torchvision cudatoolkit=10.1

RUN ldconfig /usr/local/cuda-10.1/targets/x86_64-linux/lib/stubs && \
    HOROVOD_GPU_ALLREDUCE=NCCL HOROVOD_GPU_BROADCAST=NCCL HOROVOD_WITH_TENSORFLOW=1 HOROVOD_WITH_PYTORCH=1 \
    python -m pip install --no-cache-dir horovod && \
    ldconfig

RUN python -m pip install --no-cache-dir \
    face_recognition \
    git+https://github.com/scikit-multiflow/scikit-multiflow \
    gnureadline \
    imutils \
    opencv-contrib-python \
    pyod \
    scikit-plot 

RUN python -c "from matplotlib.font_manager import _rebuild; _rebuild();"

RUN conda clean -aqy

RUN python -m pip install git+https://github.com/${REPO}/mlrun.git@${MLRUN_TAG}

# remove dask verify version mismatch warnings
RUN python -m pip install --no-cache-dir \
    blosc==1.7.0 \
    cloudpickle==1.1.1 \
    dask==2.12.0 \
    distributed==2.14.0 \
    lz4==2.2.1 \
    msgpack==1.0.0 \
    tornado==6.0.4

RUN python -m pip install urllib3==1.25.8 kubernetes==10.0.1

RUN python -m pip install lifelines scikit-plot

RUN python -m pip install git+https://github.com/${REPO}/mlutils.git@development