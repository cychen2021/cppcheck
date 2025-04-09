FROM mcr.microsoft.com/devcontainers/base:noble

RUN adduser vscode sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER vscode
ARG BUILD_DIR=/tmp/build
RUN mkdir ${BUILD_DIR}
WORKDIR ${BUILD_DIR}

RUN rm -rf /var/lib/apt/lists/*
RUN wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -
RUN echo "deb http://apt.llvm.org/noble/ llvm-toolchain-noble main" | sudo tee -a /etc/apt/sources.list && \
    echo "deb-src http://apt.llvm.org/noble/ llvm-toolchain-noble main" | sudo tee -a /etc/apt/sources.list
RUN sudo apt-get update && sudo apt-get install -y \
    software-properties-common \
    git \
    curl \
    wget \
    build-essential \
    cmake \
    ninja-build \
    clangd \
    lldb-19 \
    clang-format-19 \
    clang-tidy-19 \
    libboost-all-dev \
    vim \
    git-lfs \
    libzstd-dev \
    zstd \
    libsqlite3-dev \
    ccache \
    libomp-15-dev

RUN sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-19 100 && \
    sudo update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-19 100 && \
    sudo update-alternatives --install /usr/bin/clang-format clang-format /usr/bin/clang-format-19 100 && \
    sudo update-alternatives --install /usr/bin/clang-tidy clang-tidy /usr/bin/clang-tidy-19 100 && \
    sudo update-alternatives --install /usr/bin/lldb lldb /usr/bin/lldb-19 100 && \
    sudo update-alternatives --install /usr/bin/llvm-config llvm-config /usr/bin/llvm-config-19 100

RUN case $(uname -m) in \
		x86_64) wget -O cmake.sh "https://github.com/Kitware/CMake/releases/download/v3.31.4/cmake-3.31.4-linux-x86_64.sh" ;; \
		aarch64) wget -O cmake.sh "https://github.com/Kitware/CMake/releases/download/v3.31.4/cmake-3.31.4-linux-aarch64.sh" ;; \	
	esac;
RUN sudo sh cmake.sh --skip-license --prefix=/usr

# Needed by the CMake extension
RUN sudo add-apt-repository ppa:dotnet/backports
RUN sudo apt-get update && sudo apt-get install -y dotnet-runtime-6.0

RUN (type -p wget >/dev/null || (sudo apt update && sudo apt-get install wget -y)) \
	&& sudo mkdir -p -m 755 /etc/apt/keyrings \
        && out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
        && cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
	&& sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
	&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
	&& sudo apt update \
	&& sudo apt install gh -y

ENV EXTRA_DIR=/extra
RUN sudo mkdir -p ${EXTRA_DIR} && sudo chown -R vscode:vscode ${EXTRA_DIR} && sudo chmod -R 777 ${EXTRA_DIR}

RUN case $(uname -m) in \
        x86_64) wget -O miniconda.sh "https://repo.anaconda.com/miniconda/Miniconda3-py312_25.1.1-2-Linux-x86_64.sh" ;; \
        aarch64) wget -O miniconda.sh "https://repo.anaconda.com/miniconda/Miniconda3-py312_25.1.1-2-Linux-aarch64.sh" ;; \
    esac
ENV CONDA_DIR=${EXTRA_DIR}/miniconda3
RUN bash miniconda.sh -b -p ${CONDA_DIR}
RUN ${CONDA_DIR}/bin/conda create -n py313 python=3.13 -y
RUN ${CONDA_DIR}/bin/conda init
RUN echo "conda activate py313" >> ~/.bashrc
ENV PATH=${CONDA_DIR}/envs/py313/bin:${CONDA_DIR}/bin:${PATH}
RUN ${CONDA_DIR}/bin/conda install gh --channel conda-forge
RUN ${CONDA_DIR}/bin/python -m pip install pipx
RUN ${CONDA_DIR}/bin/pipx ensurepath
RUN sudo ${CONDA_DIR}/bin/pipx install --global 'python-lsp-server[all]'
RUN sudo update-alternatives --install /usr/bin/conda conda ${CONDA_DIR}/bin/conda 100
RUN sudo update-alternatives --install /usr/bin/pipx pipx ${CONDA_DIR}/bin/pipx 100

RUN sudo git config --global core.editor "vim"

RUN sudo rm -rf ${BUILD_DIR}

