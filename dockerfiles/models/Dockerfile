ARG MLRUN_BASE_IMAGE=mlrun/ml-base:unstable-core

FROM ${MLRUN_BASE_IMAGE}

ARG OMPI=4.1.0

# Install Open MPI
RUN mkdir /tmp/openmpi && \
    cd /tmp/openmpi && \
    curl -o openmpi-${OMPI}.tar.gz https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-${OMPI}.tar.gz && \
    tar zxf openmpi-${OMPI}.tar.gz && \
    cd openmpi-${OMPI} && \
    ./configure --enable-orterun-prefix-by-default && \
    make -j`nproc` all && \
    make install && \
    ldconfig && \
    rm -rf /tmp/openmpi

ENV OMPI_ALLOW_RUN_AS_ROOT=1
ENV OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1

# pytorch caused cuda download, this guarantees cpu version:
RUN python -m pip install torch==1.7.0+cpu torchvision==0.8.1+cpu \
    -f https://download.pytorch.org/whl/torch_stable.html

RUN python -m pip install -U 'intel-tensorflow==2.2.0' mxnet

RUN HOROVOD_WITH_TENSORFLOW=1 HOROVOD_WITH_PYTORCH=1 \
    python -m pip install 'horovod~=0.20.0'

# When running mpijob the launcher trigger the worker with something like PATH=/usr/local/bin:$PATH so we need to make
# sure the python in /usr/local/bin is pointing to the conda env python (which "knows" horovod)
# TODO: find nicer solution
RUN ln -sf /opt/conda/bin/python3 /usr/local/bin/python3

COPY ./dockerfiles/models/requirements.txt ./models-image-requirements.txt

RUN python -m pip install -r models-image-requirements.txt
