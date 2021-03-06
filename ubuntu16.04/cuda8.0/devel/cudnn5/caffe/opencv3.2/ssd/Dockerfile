FROM nvidia/cuda:8.0-cudnn5-devel-ubuntu16.04
MAINTAINER chenmingqiang <chenmingqiang86@gmail.com>

# Install python2.7
RUN chmod 777 /tmp && apt-get update && apt-get upgrade -y &&  apt-get install -y \
    build-essential \
    ca-certificates \
    wget \
    git \
    unzip \
    cmake \
    curl \
    vim \
    python2.7 \
    python2.7-dev \
    python-pip \
    pkg-config \
    libgtk2.0-dev \
    libavcodec-dev \
    libavformat-dev \
    libswscale-dev \
    libtbb2 \
    libtbb-dev \
    libjpeg-dev \
    libpng-dev \
    libtiff-dev \
    libjasper-dev \
    libdc1394-22-dev \
    && apt-get autoremove \
    && apt-get clean

# Install NCCL for multi-GPU communication
RUN wget https://github.com/NVIDIA/nccl/releases/download/v1.2.3-1%2Bcuda8.0/libnccl1_1.2.3-1.cuda8.0_amd64.deb && \
  dpkg -i libnccl1_1.2.3-1.cuda8.0_amd64.deb && \
  rm libnccl1_1.2.3-1.cuda8.0_amd64.deb && \
  wget https://github.com/NVIDIA/nccl/releases/download/v1.2.3-1%2Bcuda8.0/libnccl-dev_1.2.3-1.cuda8.0_amd64.deb && \
  dpkg -i libnccl-dev_1.2.3-1.cuda8.0_amd64.deb && \
  rm libnccl-dev_1.2.3-1.cuda8.0_amd64.deb

WORKDIR /
RUN pip install --upgrade pip && pip install numpy
RUN wget https://github.com/opencv/opencv/archive/3.2.0.zip -O opencv3.zip && \
    unzip -q opencv3.zip && mv /opencv-3.2.0 /opencv
RUN wget https://github.com/opencv/opencv_contrib/archive/3.2.0.zip -O opencv_contrib3.zip && \
    unzip -q opencv_contrib3.zip && mv /opencv_contrib-3.2.0 /opencv_contrib
RUN mkdir /opencv/build
WORKDIR /opencv/build
RUN cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D BUILD_PYTHON_SUPPORT=ON \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D INSTALL_C_EXAMPLES=OFF \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D OPENCV_EXTRA_MODULES_PATH=/opencv_contrib/modules \
    -D BUILD_EXAMPLES=ON \
    -D BUILD_NEW_PYTHON_SUPPORT=OFF \
    -D WITH_IPP=OFF \
    -D WITH_QT=OFF \
    -D CUDA_CUDA_LIBRARY=/usr/local/cuda-8.0/targets/x86_64-linux/lib/stubs/libcuda.so \
    -D WITH_V4L=ON ..
RUN make -j4 && make install && ldconfig
RUN cd / && rm -rf /opencv /opencv_contrib

# Install caffe dependencies
RUN apt-get install -y \
  libatlas-base-dev \
  libatlas-dev \
  libboost-all-dev \
  libprotobuf-dev \
  libgoogle-glog-dev \
  libgflags-dev \
  protobuf-compiler \
  libhdf5-dev \
  libleveldb-dev \
  liblmdb-dev \
  libsnappy-dev \
  # python-numpy \
  gfortran > /dev/null


WORKDIR /root
RUN pip install -U pip && cd /root && git clone https://github.com/weiliu89/caffe.git && \
    cd caffe && git checkout ssd && cat python/requirements.txt | xargs -n1 pip install && \ 
    pip install -v thrift==0.9.3 && apt-get install -y supervisor && pip install opencv-python

RUN cd /root/caffe && \
# Make and move into build directory
  mkdir build && cd build && \
# CMake
  cmake .. && \
# Make
  make -j"$(nproc)" all && \
  make install

# Add to Python path
ENV PYTHONPATH=/root/caffe/python:$PYTHONPATH

# Add caffe to path
ENV PATH=/root/caffe/build/tools:$PATH

# Set ~/caffe as working directory
WORKDIR /root/caffe

RUN apt-get install -y supervisor

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
