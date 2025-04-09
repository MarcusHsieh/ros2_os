# For Windows
ARG BASE_IMAGE=osrf/ros:humble-desktop

# For Mac
# ARG BASE_IMAGE=arm64v8/ros:humble-ros-base-jammy

FROM ${BASE_IMAGE}


# Add ubuntu user with same UID and GID as your host system, if it doesn't already exist
# Since Ubuntu 24.04, a non-root user is created by default with the name vscode and UID=1000
ARG USERNAME=ubuntu
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN if ! id -u $USER_UID >/dev/null 2>&1; then \
        groupadd --gid $USER_GID $USERNAME && \
        useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME; \
    fi
# Add sudo support for the non-root user
RUN apt-get update && \
    apt-get install -y sudo && \
    echo "$USERNAME ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME

# Switch from root to user
USER $USERNAME

# Add user to video group to allow access to webcam
RUN sudo usermod --append --groups video $USERNAME

SHELL [ "/bin/bash" , "-c" ]

# Upgrade all packages
RUN sudo apt update && sudo apt upgrade -y

# Install Git
RUN sudo apt install -y git

# Install essential packages
RUN sudo apt install -y wget

# Rosdep update
RUN rosdep update

# Create overlay workspace # might need to change WORKDIR to remove the /root/ros_ws/src part
WORKDIR /root/ros_ws/src
COPY packages.repos .

RUN vcs import < packages.repos; \
    cd ..; \
    rosdep install --from-paths src --ignore-src -r -y; \
    source /opt/ros/${ROS_DISTRO}/setup.bash; \
    colcon build;

# Add sourcing ROS setup.bash to .bashrc
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> ~/.bashrc