# Base image
FROM ubuntu:22.04

# Set non-interactive frontend to suppress prompts
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    python3-venv \
    python3-dev \
    python3-pip \
    openmpi-bin \
    libboost-all-dev \
    fftw3-dev \
    libfftw3-mpi-dev \
    git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /home/stekajack

# Create and activate a virtual environment
RUN python3 -m venv /home/stekajack/venv && \
    /home/stekajack/venv/bin/pip install --upgrade pip

# Add the virtual environment to PATH
ENV PATH="/home/stekajack/venv/bin:$PATH"
ENV VIRTUAL_ENV="/home/stekajack/venv"

# Install Python packages in the virtual environment
RUN pip install --no-cache-dir \
    numpy \
    scipy \
    cython \
    matplotlib \
    pandas \
    igraph \
    jupyterhub==3.1.1 \
    git+https://github.com/katringoogoo/batchspawner.git@1.2.0+auth_fix \
    nbclassic==0.3.7 \
    jupyterlab-system-monitor \
    jupyterlab-git \
    jupyterlab-widgets

# Clone the custom Espresso repository
RUN git clone --branch sw_beta_MPI_wip https://github.com/stekajack/espresso_patched.git espresso

# Build Espresso
WORKDIR /home/stekajack/espresso
RUN mkdir build && cd build && \
    cmake .. -DESPRESSO_BUILD_WITH_CUDA=OFF -DESPRESSO_BUILD_WITH_NLOPT=ON && \
    make -j$(nproc)

# Clone and Install Pressomancy
WORKDIR /home/stekajack
RUN git clone https://github.com/stekajack/pressomancy.git && \
    cd pressomancy && \
    pip install -e .
RUN mkdir DATA
RUN mkdir UPLOAD_VIEW

RUN git clone https://github.com/stekajack/espressoSq.git
WORKDIR /home/stekajack/espressoSq
RUN mkdir build && cd build && \
    cmake .. && \
    make -j$(nproc)
RUN python3 setup.py build_ext --inplace
WORKDIR /home/stekajack

# Cleanup
RUN rm -rf /var/cache/* /tmp/* /var/log/* /usr/share/doc/*
RUN apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/*

# Set environment variables for Espresso
ENV PYTHONPATH="/home/stekajack/espresso/build/src/python"
ENV ESPRESSOPATH="/home/stekajack/espresso/build/"
ENV OMPI_ALLOW_RUN_AS_ROOT=1
ENV OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1

# (Optional) Append environment settings to the bashrc
RUN echo 'export ESPRESSOPATH="/home/stekajack/espresso/build/"' >> /home/stekajack/.bashrc
EXPOSE 8888
CMD ["jupyterhub-singleuser", "--ip=0.0.0.0", "--port=8888", "--NotebookApp.token=''", "--NotebookApp.password=''"]