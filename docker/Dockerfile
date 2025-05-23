FROM ros:noetic-perception-focal

# Set the environment variable to non-interactive
ENV DEBIAN_FRONTEND=noninteractive

# Install necessary packages
RUN apt-get update && \
    apt-get install -y build-essential nano \
    ros-noetic-desktop-full net-tools \
    tmux \
    iputils-ping && \
    rm -rf /var/lib/apt/lists/*
    
#forgot a few stuff    
RUN apt-get update && \
    apt-get install -y net-tools iproute2 ros-noetic-plotjuggler* ros-noetic-foxglove-bridge ros-noetic-turtlesim* gedit    

# Install additional dependencies for Cartographer
RUN apt-get install -y \
    clang \
    cmake \
    g++ \
    git \
    google-mock \
    libboost-all-dev \
    libcairo2-dev \
    libceres-dev \
    libcurl4-openssl-dev \
    libeigen3-dev \
    libgflags-dev \
    libgoogle-glog-dev \
    liblua5.2-dev \
    libsuitesparse-dev \
    lsb-release \
    ninja-build \
    python3-sphinx \
    stow libgmock-dev protobuf-compiler \
    python3-wstool python3-rosdep wget

# Create a non-root user named 'robotics' with home directory
RUN useradd -m -s /bin/bash robotics && \
    mkdir -p /home/robotics/data && \
    mkdir -p /home/robotics/catkin_ws/src && \
    chown -R robotics:robotics /home/robotics

# Install Cartographer ROS
ARG CARTOGRAPHER_VERSION=master
ARG CARTOGRAPHER_SHA=LATEST
RUN mkdir -p catkin_ws/src
RUN git clone https://github.com/cartographer-project/cartographer_ros.git
RUN mv cartographer_ros/cartographer_ros_msgs catkin_ws/src && \
    mv cartographer_ros/cartographer_rviz catkin_ws/src && \
    mv cartographer_ros/cartographer_ros catkin_ws/src
RUN CARTOGRAPHER_SHA=$CARTOGRAPHER_SHA \
    CARTOGRAPHER_VERSION=$CARTOGRAPHER_VERSION \
    cartographer_ros/scripts/prepare_catkin_workspace.sh && \
    sed -i -e "s%<depend>libabsl-dev</depend>%<\!--<depend>libabsl-dev</depend>-->%g" catkin_ws/src/cartographer/package.xml
RUN cartographer_ros/scripts/install_debs.sh
RUN /catkin_ws/src/cartographer/scripts/install_abseil.sh
RUN cartographer_ros/scripts/install.sh --pkg cartographer && \
    cartographer_ros/scripts/install.sh --pkg cartographer --make-args test
RUN cartographer_ros/scripts/install.sh --pkg cartographer_ros_msgs && \
    cartographer_ros/scripts/install.sh --pkg cartographer_ros_msgs \
        --catkin-make-args run_tests && \
    cartographer_ros/scripts/catkin_test_results.sh build_isolated/cartographer_ros_msgs
RUN cartographer_ros/scripts/install.sh --pkg cartographer_ros && \
    cartographer_ros/scripts/install.sh --pkg cartographer_ros \
        --catkin-make-args run_tests && \
    cartographer_ros/scripts/catkin_test_results.sh build_isolated/cartographer_ros
RUN cartographer_ros/scripts/install.sh --pkg cartographer_rviz && \
    cartographer_ros/scripts/install.sh --pkg cartographer_rviz \
        --catkin-make-args run_tests && \
    cartographer_ros/scripts/catkin_test_results.sh build_isolated/cartographer_rviz
RUN rm -rf /var/lib/apt/lists/*
# A BTRFS bug may prevent us from cleaning up these directories.
# https://btrfs.wiki.kernel.org/index.php/Problem_FAQ#I_cannot_delete_an_empty_directory
RUN rm -rf cartographer_ros catkin_ws || true

# Switch to the robotics user
USER robotics
WORKDIR /home/robotics

# Source ROS environment for the user
RUN echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc && \
    echo "source /home/robotics/catkin_ws/devel/setup.bash" >> ~/.bashrc && \
    echo "source /opt/cartographer_ros/setup.bash" >> ~/.bashrc

# Default command
CMD ["bash"]

