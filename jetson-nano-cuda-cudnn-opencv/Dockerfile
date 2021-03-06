FROM balenalib/jetson-nano-ubuntu:bionic as buildstep

WORKDIR /usr/src/app

# Files from the SDK
COPY ./cuda-repo-l4t-10-0-local-10.0.326_1.0-1_arm64.deb .
COPY ./libcudnn7_7.6.3.28-1+cuda10.0_arm64.deb .
COPY ./libcudnn7-dev_7.6.3.28-1+cuda10.0_arm64.deb .
COPY ./libcudnn7-doc_7.6.3.28-1+cuda10.0_arm64.deb .

ENV DEBIAN_FRONTEND noninteractive

## Install runtime & build libraries and build opencv
RUN \
    dpkg -i cuda-repo-l4t-10-0-local-10.0.326_1.0-1_arm64.deb && \
    apt-key add /var/cuda-repo-10-0-local-10.0.326/*.pub && \
    dpkg -i libcudnn7_7.6.3.28-1+cuda10.0_arm64.deb \
    libcudnn7-dev_7.6.3.28-1+cuda10.0_arm64.deb \
    libcudnn7-doc_7.6.3.28-1+cuda10.0_arm64.deb && \
    apt-get update && \
    apt-get install cuda-compiler-10-0 \
    cuda-samples-10-0 \
    lbzip2 \
    git wget unzip \
    cmake automake build-essential \
    autoconf libtool \
    libgtk2.0-dev pkg-config \
    libavcodec-dev \
    libgstreamer1.0-0 \
    gstreamer1.0-plugins-base \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-plugins-ugly \
    gstreamer1.0-libav \
    gstreamer1.0-doc \
    gstreamer1.0-tools \
    libgstreamer1.0-dev \
    libgstreamer-plugins-base1.0-dev \
    libjpeg-dev \
    libpng-dev \
    libtiff-dev \
    ffmpeg \
    libdc1394-22-dev -y --no-install-recommends && \
    rm -rf ./*.deb && \
    dpkg --remove cuda-repo-l4t-10-0-local-10.0.326 && \
    dpkg -P cuda-repo-l4t-10-0-local-10.0.326 && \
    echo "/usr/lib/aarch64-linux-gnu/tegra" > /etc/ld.so.conf.d/nvidia-tegra.conf \
    && ldconfig && \
    wget https://github.com/opencv/opencv/archive/3.4.3.zip && \
    unzip 3.4.3.zip && rm 3.4.3.zip

RUN \
    wget https://github.com/opencv/opencv_contrib/archive/3.4.3.zip -O opencv_modules.3.4.3.zip && \
    unzip opencv_modules.3.4.3.zip && rm opencv_modules.3.4.3.zip && \
    mkdir -p opencv-3.4.3/build && cd opencv-3.4.3/build && \
    cmake -D WITH_CUDA=ON -D CUDA_ARCH_BIN="5.3"  -D BUILD_LIST=cudev,highgui,videoio,video,cudaimgproc,ximgproc -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-3.4.3/modules -D CMAKE_INSTALL_PREFIX=/usr/local CMAKE_BUILD_TYPE=Release -D WITH_GSTREAMER=ON -D WITH_GSTREAMER_0_10=OFF -D WITH_CUDA=OFF -D WITH_TBB=ON -D WITH_LIBV4L=ON WITH_FFMPEG=ON .. && make -j8 && make install && \
    cp /usr/src/app/opencv-3.4.3/build/bin/opencv_version /usr/src/app/ && \
    cd /usr/src/app/ && rm -rf /usr/src/app/opencv-3.4.3 && \
    rm -rf /usr/src/app/opencv_contrib-3.4.3

# Free up a little space
RUN rm -rf /usr/local/cuda-10.0/doc

# ------------------------------------------------------------------

FROM balenalib/jetson-nano-ubuntu:bionic as final

WORKDIR /usr/src/app

COPY --from=buildstep /usr/local/cuda-10.0 /usr/local/cuda-10.0

# Minimum CUDA runtime libraries
COPY --from=buildstep /usr/lib/aarch64-linux-gnu /usr/lib/aarch64-linux-gnu

# OpenCV runtime libraries
COPY --from=buildstep /usr/local/lib /usr/local/lib

COPY --from=buildstep /usr/local/include/opencv2 /usr/local/include/opencv2

# CUDNN
COPY --from=buildstep /usr/include/cudnn.h /usr/include/cudnn.h
COPY --from=buildstep /etc/alternatives/libcudnn* /etc/alternatives/
COPY --from=buildstep /usr/include/aarch64-linux-gnu/cudnn_v7.h /usr/include/aarch64-linux-gnu/cudnn_v7.h

# set paths
ENV CUDA_HOME=/usr/local/cuda-10.0/
ENV LD_LIBRARY_PATH=${CUDA_HOME}/lib64
ENV PATH=${CUDA_HOME}/bin:${PATH}
ENV UDEV=1
    
COPY ./nvidia_drivers.tbz2 .
COPY ./config.tbz2 .

ENV DEBIAN_FRONTEND noninteractive

# Prepare minimum of runtime libraries
RUN apt-get update && apt-get install -y lbzip2 pkg-config && \
    tar xjf nvidia_drivers.tbz2 -C / && \
    tar xjf config.tbz2 -C / --exclude=etc/hosts --exclude=etc/hostname && \
    echo "/usr/lib/aarch64-linux-gnu/tegra" > /etc/ld.so.conf.d/nvidia-tegra.conf && ldconfig && \
    rm -rf *.tbz2


CMD [ "sleep", "infinity" ]
