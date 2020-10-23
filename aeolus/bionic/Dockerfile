FROM nvidia/cuda:10.2-cudnn7-devel-ubuntu18.04

# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
    ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    apt-get update && \
    apt-get install -q -y --no-install-recommends tzdata && \
    rm -rf /var/lib/apt/lists/*

# install packages
RUN apt-get update && apt-get install -q -y --no-install-recommends \
    dirmngr \
    liblapack-dev \
    libblas-dev \
    libgl1-mesa-dev \
    libxt-dev \
    libflann-dev \
    git-all \
    gnupg2 \
    tar \
    wget \
    && rm -rf /var/lib/apt/lists/*

# setup keys
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# setup sources.list
RUN echo "deb http://packages.ros.org/ros/ubuntu bionic main" > /etc/apt/sources.list.d/ros1-latest.list

# setup environment
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

ENV ROS_DISTRO melodic

# install ros packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-$ROS_DISTRO-ros-core \
    && rm -rf /var/lib/apt/lists/*

# workaround for yak looking for libcuda.so.1 instead of libcuda.so
RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1

WORKDIR /vtk_build
RUN git clone --branch v7.1.1 https://github.com/Kitware/VTK.git
RUN cmake -DCMAKE_BUILD_TYPE=Release /vtk_build/VTK && \
    make -j$(nproc) && \
    make install && \
    cd .. && \
    rm -rf /vtk_build

WORKDIR /pcl_build
RUN git clone --branch pcl-1.9.1 https://github.com/PointCloudLibrary/pcl.git
RUN cmake -DCMAKE_BUILD_TYPE=MinSizeRel -DBUILD_surface_on_nurbs:BOOL=ON /pcl_build/pcl && \
    make -j$(nproc) && \
    make install && \
    cd .. && \
    rm -rf /pcl_build    

# setup entrypoint
COPY ./ros_entrypoint.sh /
ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["bash"]