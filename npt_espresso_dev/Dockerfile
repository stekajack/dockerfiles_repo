# Base image
FROM ubuntu:24.04

# Set non-interactive frontend to suppress prompts
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    cython3 \ 
    python3-venv \
    python3-dev \
    python3-pip \
    openmpi-bin \
    libboost-all-dev \
    libfftw3-dev \
    libfftw3-mpi-dev \
    git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set up a user
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
    scipy

# Clone the custom Espresso repository
RUN git clone --branch npt-redev https://github.com/hidekb/espresso.git espresso

# Build Espresso
WORKDIR /home/stekajack/espresso
RUN mkdir build && cd build && \
    cmake .. -DESPRESSO_BUILD_WITH_CUDA=OFF -DESPRESSO_BUILD_WITH_NLOPT=ON && \
    make -j$(nproc)

# Clone and Install Pressomancy
WORKDIR /home/stekajack
RUN git clone --branch kdtree_wip https://github.com/stekajack/pressomancy.git && \
    cd pressomancy && \
    pip install -e .
RUN mkdir DATA
RUN mkdir UPLOAD_VIEW

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
# Ensure the final user has full control of its home
# Keep the container running
ENTRYPOINT ["/bin/bash", "-c", "tail -f /dev/null"]