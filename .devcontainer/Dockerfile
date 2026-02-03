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
RUN apt-get update && \
    apt-get install -y curl gnupg2 lsb-release && \
    curl -sSL 'http://repo.ros2.org/repos.key' | apt-key add - && \
    sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' && \
    apt-get update && \
    apt-get install -y ros-noetic-desktop-full python3-rosinstall python3-rosinstall-generator python3-wstool build-essential python3-rosdep && \
    rosdep init && \
    rosdep update

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

# Build blasfeo from source
RUN git clone https://github.com/giaf/blasfeo.git \
    && cmake -B build -S blasfeo -DTARGET=X64_AUTOMATIC \
    && cmake --build build --parallel 4 \
    && cmake --install build \
    && rm -rf blasfeo build

# Build fatrop solver from source
RUN git clone https://github.com/meco-group/fatrop.git --recursive \
    && cmake -B build -S fatrop -DBLASFEO_TARGET=X64_AUTOMATIC -DENABLE_MULTITHREADING=OFF \
    && cmake --build build --parallel 4 \
    && cmake --install build \
    && rm -rf fatrop build

# Build and install CasADi from source
RUN git clone -b 3.6.7 --recursive https://github.com/casadi/casadi.git \
    && cmake -B build -S casadi -DWITH_PYTHON=ON -DWITH_PYTHON3=ON -DWITH_OPENMP=ON -DWITH_THREAD=ON -DWITH_IPOPT=ON -DWITH_FATROP=ON -DWITH_DOC=OFF -DPYTHON_PREFIX=/root/workspace/mmdeploy/build/lib \
    && cmake --build build --parallel 4 \
    && cmake --install build \
    && rm -rf casadi build

# Build libfranka
RUN git clone -b 0.9.2 --recursive https://github.com/frankaemika/libfranka \
    && cmake -B build -S libfranka -DBUILD_TESTS=OFF \
    && cmake --build build --parallel 4 \
    && cmake --install build \
    && rm -rf libfranka build

# INSTALL THE DEPEDANCIES 
RUN git clone https://gitlab.laas.fr/msabbah/panda_torque_mpc.git /tmp/ros_ws/src/panda_torque_mpc 

RUN mkdir -p /deps_ws/src
    
RUN vcs import --recursive /deps_ws/src < /tmp/ros_ws/src/panda_torque_mpc/panda_torque_mpc.repos \
    && rm -rf /deps_ws/src/vcs_repos/tsid

# Build libs
RUN cd /deps_ws \ 
    && source /opt/ros/noetic/setup.bash \
    && apt update \
    && rosdep update --rosdistro $ROS_DISTRO \
    && rosdep install -y -i --from-paths src --rosdistro $ROS_DISTRO \
    && catkin build -j4 \
        --cmake-args \
        -DPYTHON_EXECUTABLE=/usr/bin/python \
        -DBUILD_PYTHON_INTERFACE=ON \
        -DPYTHON_SITELIB=lib/python3/dist-packages \
        -DBUILD_WITH_URDF_SUPPORT=ON \
        -DBUILD_WITH_CASADI_SUPPORT=ON \
        -DBUILD_WITH_COLLISION_SUPPORT=ON \
        -DBUILD_BENCHMARK=OFF \
        -DBUILD_BENCHMARKS=OFF \
        -DBUILD_EXAMPLES=OFF \
        -DBUILD_WITH_IPOPT=OFF \
        -DBUILD_TESTING=OFF \
        -DBUILD_UNIT_TEST=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DGENERATE_PYTHON_STUBS=OFF \
        -DCMAKE_CXX_FLAGS="-O3"

WORKDIR /root/workspace/ros_ws

RUN git clone https://gitlab.laas.fr/msabbah/panda_torque_mpc.git /root/workspace/ros_ws/src/panda_torque_mpc \ 
&& git clone https://gitlab.laas.fr/msabbah/rt-cosmik.git /root/workspace/ros_ws/src/rt-cosmik \
&& git clone https://gitlab.laas.fr/msabbah/motif_ros.git /root/workspace/ros_ws/src/motif_ros \
&& git clone https://github.com/open-mmlab/mmpose.git /root/workspace/mmpose

RUN mim install mmdet mmpose

# FOR RT-COSMIK 
RUN pip uninstall -y opencv-contrib-python opencv-python \
&& pip install opencv-contrib-python==4.7.0.68 opencv-python==4.7.0.68

RUN echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc \
    && echo "source /root/workspace/ros_ws/devel/setup.bash" >> ~/.bashrc \
    && echo "source /deps_ws/devel/setup.bash" >> ~/.bashrc \
    && echo "export LIBRARY_PATH=/opt/blasfeo/lib:$LIBRARY_PATH" >> ~/.bashrc \
    && echo "export PYTHONPATH=$PYTHONPATH:/opt/openrobots/lib/python3.8/site-packages" >> ~/.bashrc \
    && echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/openrobots/lib" >> ~/.bashrc \
    && echo "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/blasfeo/lib" >> ~/.bashrc \
    && echo "export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/opt/openrobots/lib/pkgconfig" >> ~/.bashrc \
    && echo "export CPATH=/opt/blasfeo/include:$CPATH" >> ~/.bashrc \
    && echo "export PATH=$PATH:/opt/openrobots/bin" >> ~/.bashrc 

# To open the docker in a terminal window after having it launched in vscode try 
# docker ps 
# (to get the name of the container) and then 
# docker exec -it <mycontainer> bash