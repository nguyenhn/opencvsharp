FROM ubuntu:18.04 AS build-native-env

ENV OPENCV_VERSION=4.3.0

RUN apt-get update && apt-get install -y \
    apt-transport-https \
    software-properties-common \
    wget \
    unzip \
    curl \
    ca-certificates
    #bzip2 \
    #grep sed dpkg 

# Install opencv dependencies
RUN cd ~
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    gfortran \
    libjpeg8-dev \
    libpng-dev \
    software-properties-common
RUN add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main" && apt-get update && apt-get install -y \
    libjasper1 \
    libtiff-dev \
    libavcodec-dev \
    libavformat-dev \
    libswscale-dev \
    libdc1394-22-dev \
    libxine2-dev \
    libv4l-dev

RUN cd /usr/include/linux
RUN ln -s -f ../libv4l1-videodev.h videodev.h
RUN cd ~
RUN apt-get install -y \
    libgtk2.0-dev libtbb-dev qt5-default \
    libatlas-base-dev \
    libfaac-dev \
    libmp3lame-dev \
    libtheora-dev \
    libvorbis-dev \
    libxvidcore-dev \
    libopencore-amrnb-dev \
    libopencore-amrwb-dev \
    libavresample-dev \
    x264 \
    v4l-utils

# Setup OpenCV source
RUN wget https://github.com/opencv/opencv/archive/${OPENCV_VERSION}.zip && \
    unzip ${OPENCV_VERSION}.zip && \
    rm ${OPENCV_VERSION}.zip && \
    mv opencv-${OPENCV_VERSION} opencv

# Setup opencv-contrib Source
RUN wget https://github.com/opencv/opencv_contrib/archive/${OPENCV_VERSION}.zip && \
    unzip ${OPENCV_VERSION}.zip && \
    rm ${OPENCV_VERSION}.zip && \
    mv opencv_contrib-${OPENCV_VERSION} opencv_contrib

# Build OpenCV
RUN cd opencv && mkdir build && cd build && \
    cmake \
    -D OPENCV_EXTRA_MODULES_PATH=/opencv_contrib/modules \
    -D CMAKE_BUILD_TYPE=RELEASE \
    -D BUILD_SHARED_LIBS=OFF \
    -D ENABLE_CXX11=ON \
    -D BUILD_EXAMPLES=OFF \
    -D BUILD_DOCS=OFF \
    -D BUILD_PERF_TESTS=OFF \
    -D BUILD_TESTS=OFF \
    -D BUILD_JAVA=OFF \
    -D BUILD_opencv_app=OFF \
    -D BUILD_opencv_java=OFF \
    -D BUILD_opencv_python=OFF \
    -D BUILD_opencv_ts=OFF \
    -D BUILD_opencv_js=OFF \
    -D WITH_GSTREAMER=OFF \ 
    -D OPENCV_ENABLE_NONFREE=ON \
    .. && make -j4 && make install && ldconfig

WORKDIR /

# Download OpenCvSharp
RUN git clone https://github.com/shimat/opencvsharp.git
RUN cd opencvsharp && git fetch --all --tags --prune

# Install the Extern lib.
WORKDIR /opencvsharp/src
RUN mkdir /opencvsharp/make
RUN cd /opencvsharp/make && cmake -D CMAKE_INSTALL_PREFIX=/opencvsharp/make /opencvsharp/src && make -j4 && make install
RUN ls /opencvsharp/make

FROM mcr.microsoft.com/dotnet/core/sdk:3.0 AS build-dotnet-env
COPY --from=build-native-env /opencvsharp/make/OpenCvSharpExtern/libOpenCvSharpExtern.so ./
RUN git clone https://github.com/shimat/opencvsharp.git
RUN pwd
RUN ls

# Install Build the C# part of OpenCvSharp
WORKDIR /opencvsharp/src/OpenCvSharp
RUN cd /opencvsharp/src/OpenCvSharp
RUN dotnet build -c Release -f netstandard2.0 

WORKDIR /opencvsharp/src/OpenCvSharp.Blob
RUN cd /opencvsharp/src/OpenCvSharp.Blob
RUN dotnet build -c Release -f netstandard2.0 

WORKDIR /opencvsharp/src/OpenCvSharp.Extensions
RUN cd /opencvsharp/src/OpenCvSharp.Extensions
RUN dotnet build -c Release -f netstandard2.0 

RUN mkdir /opencvsharp/build
WORKDIR /opencvsharp/build
RUN cp /libOpenCvSharpExtern.so .
RUN cp /opencvsharp/src/OpenCvSharp/bin/Release/netstandard2.0/* .
RUN cp /opencvsharp/src/OpenCvSharp.Blob/bin/Release/netstandard2.0/* .
RUN cp /opencvsharp/src/OpenCvSharp.Extensions/bin/Release/netstandard2.0/* .
RUN pwd
RUN ls





FROM mcr.microsoft.com/dotnet/core/runtime:3.0
WORKDIR /app
COPY --from=build-dotnet-env /opencvsharp/build ./
RUN pwd
RUN ls
#ENTRYPOINT ["ls"]
