FROM nvidia/cuda:12.1.1-cudnn8-devel-ubuntu22.04

SHELL ["/bin/bash", "-c"]

ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    RCUTILS_COLORIZED_OUTPUT=1 \
    CMAKE_BUILD_TYPE=RelWithDebInfo \
    ROS_DISTRO=humble \
    DEBIAN_FRONTEND=noninteractive

# robotpkg 
ADD --chmod=644 http://robotpkg.openrobots.org/packages/debian/robotpkg.gpg /robotpkg.gpg
RUN echo "deb [arch=amd64 signed-by=/robotpkg.gpg] http://robotpkg.openrobots.org/wip/packages/debian/pub jammy robotpkg" >> /etc/apt/sources.list.d/robotpkg.list
RUN echo "deb [arch=amd64 signed-by=/robotpkg.gpg] http://robotpkg.openrobots.org/packages/debian/pub jammy robotpkg" >> /etc/apt/sources.list.d/robotpkg.list

ENV PATH=/opt/openrobots/bin:${PATH}
ENV PKG_CONFIG_PATH=/opt/openrobots/lib/pkgconfig:${PKG_CONFIG_PATH}
ENV LD_LIBRARY_PATH=/opt/openrobots/lib:${LD_LIBRARY_PATH}
ENV PYTHONPATH=/usr/local/lib/python3.10/site-packages:/opt/openrobots/lib/python3.10/site-packages:${PYTHONPATH}
ENV CMAKE_PREFIX_PATH=/opt/openrobots:${CMAKE_PREFIX_PATH}


# Install locales and other necessary tools
RUN apt-get update && \
    apt-get install -y locales && \
    locale-gen en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8

RUN add-apt-repository universe \
    && apt-get update \
    && apt-get install -y --no-install-recommends curl python3 ca-certificates \
    && ROS_APT_SOURCE_VERSION="$(curl -fsSL -H 'User-Agent: docker' -H 'Accept: application/vnd.github+json' \
      https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest \
      | python3 -c 'import sys,json; print(json.load(sys.stdin)["tag_name"])')" \
    && test -n "${ROS_APT_SOURCE_VERSION}" \
    && CODENAME="$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})" \
    && curl -fsSL -o /tmp/ros2-apt-source.deb \
      "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.${CODENAME}_all.deb" \
    && dpkg -i /tmp/ros2-apt-source.deb \
    && rm -f /tmp/ros2-apt-source.deb \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
      ros-${ROS_DISTRO}-desktop \
      ros-dev-tools \
      python3-rosdep python3-colcon-common-extensions python3-vcstool \
    && rosdep init \
    && rosdep update

# Install required packages
RUN --mount=type=cache,sharing=locked,target=/var/cache/apt \
    --mount=type=cache,sharing=locked,target=/var/lib/apt \
    --mount=type=cache,sharing=locked,target=/root/.cache \
    apt-get update \
    && apt-get install -qqy --no-install-recommends \
        # general dependencies
        bash-completion \
        build-essential \
        clang-format \
        ccache \
        liburdfdom-tools \
        usbutils \
        cmake \
        gdb \
        git \
        htop \
        nano \
        iputils-ping \
        net-tools \
        sudo \
        wget \
        vim \
        v4l-utils \
        # OpenCV / GUI runtime deps (libxcb.so.1, etc.)
        libxcb1 \
        libx11-6 \
        libxext6 \
        libxrender1 \
        libxkbcommon0 \
        libxfixes3 \
        libxi6 \
        libxrandr2 \
        libxcursor1 \
        libxcomposite1 \
        libxdamage1 \
        libglib2.0-0 \
        libgl1 \
        libsm6 \
        libgtk-3-0 \
        # TensorRT CLI tools
        libnvinfer-bin \
        # gepetto-viewer 
        robotpkg-py310-qt5-gepetto-viewer-corba \
        #casadi dependencies
        gfortran \
        liblapack-dev \
        libblas-dev \
        libopenblas-dev \
        coinor-libipopt-dev \
        swig \
        pkg-config \
        libmetis-dev \
        libgmp-dev \
        # python
        python3 \
        python3-venv \
        python3-distutils \
        ipython3 \
        python3-dev \
        python-is-python3 \
        python3-flake8 \
        python3-pip \
        python3-setuptools \
        # pinocchio
        robotpkg-py310-pinocchio \
        robotpkg-pinocchio \
    && python -m pip install -U \
        torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 --index-url https://download.pytorch.org/whl/cu121 \
        argcomplete \
        pre-commit \
        quadprog \
        pyrealsense2 \
        ultralytics \
        opencv-python \
        onnxruntime \
        looseversion \
        keyboard \
        xacrodoc \
        pynput

# Backlog of the image 
RUN mkdir -p /opt/manifest; \
  python -m pip freeze > /opt/manifest/pip_freeze.txt; \
  dpkg-query -W > /opt/manifest/dpkg.txt; \
  apt-mark showmanual > /opt/manifest/apt_manual.txt

# -----------------------------------------------------------------------------
# Workspace layout (as you liked)
# -----------------------------------------------------------------------------
RUN mkdir -p /root/workspace/src /root/workspace/ros_ws
WORKDIR /root/workspace

# Source ROS 2 + openrobots env by default in interactive shells
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /root/.bashrc; \
    echo "source /etc/profile.d/openrobots.sh" >> /root/.bashrc

# -----------------------------------------------------------------------------
# Build BLASFEO, FATROP, CasADi from source (installed into /usr/local + venv)
# -----------------------------------------------------------------------------
WORKDIR /root/workspace/src

RUN git clone https://github.com/giaf/blasfeo.git; \
    cd blasfeo; \
    git checkout 9923ac8a63d710d894a3d2987d885002df69db74; \
    mkdir -p build; \
    cd build; \
    cmake .. \
      -DTARGET=X64_AUTOMATIC \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DCMAKE_BUILD_TYPE=Release; \
    make -j"$(nproc)"; \
    make install

RUN git clone https://github.com/meco-group/fatrop.git; \
    cd fatrop; \
    git checkout 45ee388750ba2b4cf7ba603203d4c12431e485fe; \
    mkdir -p build; \
    cd build; \
    cmake .. \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_TESTS=OFF; \
    make -j"$(nproc)"; \
    make install

RUN git clone https://github.com/casadi/casadi.git; \
    cd casadi; \
    git checkout 3.7.2; \
    mkdir -p build; \
    cd build; \
    cmake .. \
      -DWITH_IPOPT=ON -DWITH_BUILD_IPOPT=ON \
      -DWITH_FATROP=ON \
      -DWITH_OPENMP=ON -DWITH_THREAD=ON \
      -DWITH_BUILD_MUMPS=ON -DWITH_BUILD_METIS=ON \
      -DWITH_PYTHON=ON -DWITH_PYTHON3=ON \
      -DPYTHON_PREFIX="$(python -c 'from distutils.sysconfig import get_python_lib; print(get_python_lib())')" \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DCMAKE_BUILD_TYPE=Release; \
    make -j1 mumps-external; \
    make -j"$(nproc)"; \
    make install

WORKDIR /root/workspace

RUN echo "export PATH=$PATH:/opt/openrobots/bin" >> /root/.bashrc; \
    echo "export PYTHONPATH=$PYTHONPATH:/opt/openrobots/lib/python3.10/site-packages:/usr/local/lib/python3.10/site-packages" >> /root/.bashrc; \
    echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/openrobots/lib:/usr/local/lib" >> /root/.bashrc; \
    echo "export LIBRARY_PATH=/usr/local/lib:$LIBRARY_PATH" >> /root/.bashrc; \
    echo "export CPATH=/usr/local/include:$CPATH" >> /root/.bashrc; \
    echo "export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/opt/openrobots/lib/pkgconfig:/usr/local/lib/pkgconfig" >> /root/.bashrc

CMD ["bash"]
