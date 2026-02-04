FROM nvidia/cuda:12.1.1-cudnn8-devel-ubuntu22.04

ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-c"]

ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    RCUTILS_COLORIZED_OUTPUT=1 \
    CMAKE_BUILD_TYPE=RelWithDebInfo \
    ROS_DISTRO=humble

RUN apt-get update; \
    apt-get install -y --no-install-recommends \
      ca-certificates curl git locales lsb-release gnupg2 \
      software-properties-common \
      # python
      python3 python3-venv python3-dev python3-pip python-is-python3 python3-distutils \
      # network/debug/editor utils
      bash-completion build-essential cmake gdb htop nano vim wget \
      iputils-ping net-tools sudo usbutils v4l-utils \
      clang-format ccache \
      liburdfdom-tools \
      # OpenCV / GUI runtime deps (libxcb.so.1, etc.)
      libxcb1 libx11-6 libxext6 libxrender1 libxkbcommon0 \
      libxfixes3 libxi6 libxrandr2 libxcursor1 libxcomposite1 libxdamage1 \
      libglib2.0-0 libgl1 libsm6 libgtk-3-0 \
      # TensorRT CLI tools
      libnvinfer-bin \
      # numeric / optimization deps
      gfortran liblapack-dev libblas-dev libopenblas-dev coinor-libipopt-dev swig pkg-config \
      libmetis-dev libgmp-dev \
      # misc often-needed runtime libs
      libusb-1.0-0 udev; \
    locale-gen en_US.UTF-8; \
    rm -rf /var/lib/apt/lists/*

RUN add-apt-repository universe; \
    apt-get update; \
    apt-get install -y --no-install-recommends curl python3 ca-certificates; \
    ROS_APT_SOURCE_VERSION="$(curl -fsSL -H 'User-Agent: docker' -H 'Accept: application/vnd.github+json' \
      https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest \
      | python3 -c 'import sys,json; print(json.load(sys.stdin)["tag_name"])')"; \
    test -n "${ROS_APT_SOURCE_VERSION}"; \
    CODENAME="$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})"; \
    curl -fsSL -o /tmp/ros2-apt-source.deb \
      "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.${CODENAME}_all.deb"; \
    dpkg -i /tmp/ros2-apt-source.deb; \
    rm -f /tmp/ros2-apt-source.deb; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      ros-${ROS_DISTRO}-desktop \
      ros-dev-tools \
      python3-rosdep python3-colcon-common-extensions python3-vcstool; \
    rosdep init || true; \
    rosdep update; \
    rm -rf /var/lib/apt/lists/*


RUN apt-get update; \
    apt-get install -y --no-install-recommends lsb-release curl; \
    mkdir -p /etc/apt/keyrings; \
    curl -fsSL http://robotpkg.openrobots.org/packages/debian/robotpkg.asc \
      | tee /etc/apt/keyrings/robotpkg.asc > /dev/null; \
    echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/robotpkg.asc] http://robotpkg.openrobots.org/packages/debian/pub $(lsb_release -cs) robotpkg" \
      > /etc/apt/sources.list.d/robotpkg.list; \
    apt-get update; \
    # Package names depend on Ubuntu/Python series (22.04 is typically py310).
    # We try the common candidates to be resilient if robotpkg bumps naming.
    (apt-get install -y --no-install-recommends robotpkg-py310-pinocchio || \
     apt-get install -y --no-install-recommends robotpkg-py3-pinocchio   || \
     apt-get install -y --no-install-recommends robotpkg-pinocchio); \
    (apt-get install -y --no-install-recommends robotpkg-py310-qt5-gepetto-viewer-corba || \
     apt-get install -y --no-install-recommends robotpkg-py3-qt5-gepetto-viewer-corba   || \
     apt-get install -y --no-install-recommends robotpkg-py310-qt5-gepetto-viewer); \
    rm -rf /var/lib/apt/lists/*

# Make robotpkg environment available everywhere
RUN cat > /etc/profile.d/openrobots.sh <<'EOF'
export PATH=/opt/openrobots/bin:$PATH
export PKG_CONFIG_PATH=/opt/openrobots/lib/pkgconfig:$PKG_CONFIG_PATH
export LD_LIBRARY_PATH=/opt/openrobots/lib:$LD_LIBRARY_PATH
export CMAKE_PREFIX_PATH=/opt/openrobots:$CMAKE_PREFIX_PATH
# robotpkg python path (Ubuntu 22.04 -> python3.10)
export PYTHONPATH=/opt/openrobots/lib/python3.10/site-packages:$PYTHONPATH
EOF


# Install uv
ENV UV_NO_MODIFY_PATH=1
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
# Common install location for uv
ENV PATH="/root/.local/bin:${PATH}"

# NOTE: torch==...+cu121 needs the PyTorch extra index URL.
RUN python -m pip install --upgrade pip; \
  cat > /tmp/requirements-ml.txt <<'EOF'
asttokens==3.0.1
certifi==2026.1.4
charset-normalizer==3.4.4
colorama==0.4.6
coloredlogs==15.0.1
contourpy==1.3.2
cycler==0.12.1
decorator==5.2.1
exceptiongroup==1.3.1
executing==2.2.1
filelock==3.20.0
flatbuffers==25.12.19
fonttools==4.61.1
fsspec==2025.12.0
humanfriendly==10.0
idna==3.11
ipython==8.38.0
jedi==0.19.2
Jinja2==3.1.6
kiwisolver==1.4.9
MarkupSafe==2.1.5
matplotlib==3.10.8
matplotlib-inline==0.2.1
ml_dtypes==0.5.4
mpmath==1.3.0
networkx==3.4.2
numpy==2.2.6
nvidia-cublas-cu12==12.1.3.1
nvidia-cuda-cupti-cu12==12.1.105
nvidia-cuda-nvrtc-cu12==12.1.105
nvidia-cuda-runtime-cu12==12.1.105
nvidia-cudnn-cu12==9.1.0.70
nvidia-cufft-cu12==11.0.2.54
nvidia-curand-cu12==10.3.2.106
nvidia-cusolver-cu12==11.4.5.107
nvidia-cusparse-cu12==12.1.0.106
nvidia-nccl-cu12==2.20.5
nvidia-nvjitlink-cu12==12.9.86
nvidia-nvtx-cu12==12.1.105
onnx==1.20.1
onnxruntime-gpu==1.23.2
onnxslim==0.1.83
opencv-python==4.13.0.90
packaging==26.0
parso==0.8.5
pexpect==4.9.0
pillow==12.0.0
polars==1.37.1
polars-runtime-32==1.37.1
prompt_toolkit==3.0.52
protobuf==6.33.5
psutil==7.2.2
ptyprocess==0.7.0
pure_eval==0.2.3
Pygments==2.19.2
pyparsing==3.3.2
python-dateutil==2.9.0.post0
PyYAML==6.0.3
requests==2.32.5
scipy==1.15.3
six==1.17.0
stack-data==0.6.3
tensorrt-cu12==10.15.1.29
tensorrt_cu12_bindings==10.15.1.29
tensorrt_cu12_libs==10.15.1.29
torch==2.4.1+cu121
torchaudio==2.4.1+cu121
torchvision==0.19.1+cu121
traitlets==5.14.3
triton==3.0.0
typing_extensions==4.15.0
ultralytics==8.4.11
ultralytics-thop==2.0.18
urllib3==2.6.3
wcwidth==0.5.3
EOF

RUN python -m pip install --no-cache-dir --extra-index-url https://download.pytorch.org/whl/cu121 -r /tmp/requirements-ml.txt; \
  python -m pip install --no-cache-dir \
    argcomplete pre-commit quadprog pyrealsense2 looseversion keyboard xacrodoc pynput; \
  rm -f /tmp/requirements-ml.txt

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
