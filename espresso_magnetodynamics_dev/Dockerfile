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

WORKDIR /home/espresso

# Create and activate a virtual environment
RUN python3 -m venv /home/espresso/venv && \
    /home/espresso/venv/bin/pip install --upgrade pip

# Add the virtual environment to PATH
ENV PATH="/home/espresso/venv/bin:$PATH"
ENV VIRTUAL_ENV="/home/espresso/venv"

# Install Python packages in the virtual environment
RUN pip install --no-cache-dir \
    numpy \
    scipy \
    cython

# Clone the custom Espresso repository
RUN git clone --branch magnetodynamics_wip https://github.com/stekajack/espresso_patched.git espresso

# Build Espresso
WORKDIR /home/espresso/espresso
RUN mkdir build && cd build && \
    cmake .. -DESPRESSO_BUILD_WITH_CUDA=OFF -DESPRESSO_BUILD_WITH_NLOPT=ON && \
    make -j$(nproc)
# Cleanup
RUN rm -rf /var/cache/* /tmp/* /var/log/* /usr/share/doc/*
RUN apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/*

# Set environment variables for Espresso
ENV PYTHONPATH="/home/espresso/espresso/build/src/python"
ENV ESPRESSOPATH="/home/espresso/espresso/build/"
ENV OMPI_ALLOW_RUN_AS_ROOT=1
ENV OMPI_ALLOW_RUN_AS_ROOT_CONFIRM=1

# (Optional) Append environment settings to the bashrc
RUN echo 'export ESPRESSOPATH="/home/espresso/espresso/build/"' >> /home/espresso/.bashrc
