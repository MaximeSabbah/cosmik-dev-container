FROM openmmlab/mmdeploy:ubuntu20.04-cuda11.8-mmdeploy1.3.1

SHELL ["/bin/bash", "-c"]

ENV RCUTILS_COLORIZED_OUTPUT=1
ENV CMAKE_BUILD_TYPE=RelWithDebInfo

ENV DEBIAN_FRONTEND=noninteractive

# robotpkg for gepetto-viewer
ADD --chmod=644 http://robotpkg.openrobots.org/packages/debian/robotpkg.gpg /robotpkg.gpg
RUN echo "deb [arch=amd64 signed-by=/robotpkg.gpg] http://robotpkg.openrobots.org/wip/packages/debian/pub focal robotpkg" >> /etc/apt/sources.list.d/robotpkg.list
RUN echo "deb [arch=amd64 signed-by=/robotpkg.gpg] http://robotpkg.openrobots.org/packages/debian/pub focal robotpkg" >> /etc/apt/sources.list.d/robotpkg.list

# INSTALLING ROS NOETIC
# Install locales and other necessary tools
RUN apt-get update && \
    apt-get install -y locales && \
    locale-gen en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8

# Install necessary dependencies and ROS Noetic
RUN apt-get update && apt-get install -y --no-install-recommends \
      curl gnupg2 lsb-release ca-certificates \
 && rm -f /usr/share/keyrings/ros-archive-keyring.gpg \
 && curl -fsSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
      -o /usr/share/keyrings/ros-archive-keyring.gpg \
 && echo "deb [signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" \
      > /etc/apt/sources.list.d/ros-latest.list \
 && apt-get update \
 && apt-get install -y --no-install-recommends \
      ros-noetic-desktop-full \
      python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential \
 && rosdep init \
 && rosdep update


# Install required packages
RUN --mount=type=cache,sharing=locked,target=/var/cache/apt \
    --mount=type=cache,sharing=locked,target=/var/lib/apt \
    --mount=type=cache,sharing=locked,target=/root/.cache \
    apt-get update \
    && apt-get upgrade -y \
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
        # gepetto-viewer 
        robotpkg-py38-qt5-gepetto-viewer-corba \
        #casadi dependencies
        # ipopt
        gfortran \
        liblapack-dev \
        libblas-dev \
        coinor-libipopt-dev \
        swig \
        ipython3 \
        python3-dev \
        python3-numpy \
        python3-scipy \
        python3-matplotlib \
        # libfranka dependencies
        libpoco-dev \
        libeigen3-dev \
        # python dependencies
        python-is-python3 \
        python3-flake8 \
        python3-pip \
        python3-setuptools \
        # ros dependencies
        python3-vcstool \
        python3-catkin-tools \
        # deps eigenpy/hppfcl/pinocchio (source build)
        libeigen3-dev libboost-all-dev libboost-python-dev \
        liburdfdom-dev libconsole-bridge-dev libtinyxml2-dev \
        pybind11-dev \
        libassimp-dev liboctomap-dev libqhull-dev \
    && python -m pip install -U \
        argcomplete \
        pre-commit \
        quadprog \
        pyrealsense2 \
        onnxruntime \
        looseversion \
        keyboard \
        xacrodoc \
        pynput

# -----------------------------------------------------------------------------
# Workspace layout
# -----------------------------------------------------------------------------
RUN mkdir -p /root/workspace/deps /root/workspace/ros_ws
WORKDIR /root/workspace
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /root/.bashrc

# -----------------------------------------------------------------------------
# Build BLASFEO, FATROP, CasADi from source (installed into /usr/local)
# -----------------------------------------------------------------------------
WORKDIR /root/workspace/deps

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
    make install; \
    ldconfig

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
    make install; \
    ldconfig

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
    make install; \
    ldconfig

# -----------------------------------------------------------------------------
# Build eigenpy from source (pinned) -> installs /usr/local/lib{,64}/cmake/eigenpy + libeigenpy.so
# -----------------------------------------------------------------------------
RUN git clone https://github.com/stack-of-tasks/eigenpy.git; \
    cd eigenpy; \
    git checkout v3.12.0; \
    mkdir -p build; \
    cd build; \
    cmake .. \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_TESTING=OFF \
      -DPython_EXECUTABLE=/usr/bin/python \
      -DCMAKE_PREFIX_PATH="/usr/local"; \
    make -j"$(nproc)"; \
    make install; \
    ldconfig

# -----------------------------------------------------------------------------
# Build Pinocchio from source WITH CasADi + Python bindings + HPP-FCL (pinned) + example_robot_data for cosmik
# -----------------------------------------------------------------------------
RUN git clone https://github.com/coal-library/coal.git; \
    cd coal; \
    git checkout v3.0.2; \
    mkdir -p build; \
    cd build; \
    cmake .. \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_TESTING=OFF \
      -DBUILD_PYTHON_INTERFACE=ON \
      -DCOAL_BACKWARD_COMPATIBILITY_WITH_HPP_FCL=ON \
      -DPYTHON_EXECUTABLE=/usr/bin/python \
      -DPython_EXECUTABLE=/usr/bin/python \
      -Deigenpy_DIR=/usr/local/lib/cmake/eigenpy \
      -DCMAKE_PREFIX_PATH="/usr/local"; \
    make -j"$(nproc)"; \
    make install; \
    ldconfig

RUN git clone --recursive https://github.com/stack-of-tasks/pinocchio.git; \
    cd pinocchio; \
    git checkout v3.9.0; \
    git submodule update --init --recursive; \
    mkdir -p build; \
    cd build; \
    cmake .. \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_TESTING=OFF \
      -DBUILD_EXAMPLES=OFF \
      -DBUILD_BENCHMARK=OFF \
      -DBUILD_PYTHON_INTERFACE=ON \
      -DBUILD_WITH_CASADI_SUPPORT=ON \
      -DBUILD_WITH_COLLISION_SUPPORT=ON \
      -DPython_EXECUTABLE=/usr/bin/python \
      -Deigenpy_DIR=/usr/local/lib/cmake/eigenpy \
      -DCMAKE_PREFIX_PATH="/usr/local"; \
    make -j"$(nproc)"; \
    make install; \
    ldconfig

WORKDIR /root/workspace

RUN git clone https://github.com/open-mmlab/mmpose.git /root/workspace/mmpose

RUN mim install mmdet mmpose

# FOR RT-COSMIK 
RUN pip uninstall -y opencv-contrib-python opencv-python \
&& pip install opencv-contrib-python==4.7.0.68 opencv-python==4.7.0.68

RUN echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc \
    && echo "export PATH=$PATH:/opt/openrobots/bin" >> ~/.bashrc \
    && echo "export PYTHONPATH=$PYTHONPATH:/opt/openrobots/lib/python3.8/site-packages:/usr/local/lib/python3.8/site-packages" >> ~/.bashrc \
    && echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/openrobots/lib:/usr/local/lib" >> ~/.bashrc \
    && echo "export LIBRARY_PATH=/usr/local/lib:$LIBRARY_PATH" >> ~/.bashrc \
    && echo "export CPATH=/usr/local/include:$CPATH" >> ~/.bashrc \
    && echo "export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/opt/openrobots/lib/pkgconfig:/usr/local/lib/pkgconfig" >> ~/.bashrc

# To open the docker in a terminal window after having it launched in vscode try 
# docker ps 
# (to get the name of the container) and then 
# docker exec -it <mycontainer> bash