FROM ros:melodic-perception-bionic

# ifm3d git release tag to build
ARG vtag

# if defined, we run unit tests when building ifm3d
ARG run_tests

# make -j $ncores
ARG ncores=4

# if you are running unit tests against a camera at
# a different IP, set that here.
ENV IFM3D_IP 192.168.0.69

#####################################################
#####################################################
ENV DEBIAN_FRONTEND noninteractive

# See:
# http://answers.ros.org/question/325039/apt-update-fails-cannot-install-pkgs-key-not-working/
RUN apt-key del 421C365BD9FF1F717815A3895523BAEEB01FA116
RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

RUN apt-get update && apt-get -y upgrade
RUN apt-get update && \
    apt-get install -y jq \
                       libgoogle-glog-dev \
                       libxmlrpc-c++8-dev \
                       python-dev \
                       python3-dev \
                       python-pytest \
                       python3-pytest \
                       python-numpy \
                       python3-numpy \
                       locales \
                       sudo

RUN mkdir -p /opt && \
    cd /opt && \
    git clone https://github.com/pybind/pybind11 && \
    cd pybind11 && \
    git checkout tags/v2.3.0 && \
    mkdir build && \
    cd build && \
    cmake -DPYBIND11_TEST=OFF .. && \
    make && \
    make install

RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    locale-gen en_US.UTF-8 && \
    /usr/sbin/update-locale LANG=en_US.UTF-8

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

RUN id ifm 2>/dev/null || useradd --uid 30000 --create-home -s /bin/bash -U ifm
RUN echo "ifm ALL=(ALL) NOPASSWD: ALL" | tee -a /etc/sudoers

USER ifm
WORKDIR /home/ifm
RUN /bin/bash -c 'mkdir src && \
    cd src && \
    source /opt/ros/melodic/setup.bash && \
    git clone https://github.com/ifm/ifm3d.git && \
    cd ifm3d && \
    if [ "x$vtag" = "x" ]; then echo "Building from master"; else git checkout tags/$vtag; fi && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_INSTALL_PREFIX=/usr -DBUILD_MODULE_OPENCV=ON -DBUILD_MODULE_PCICCLIENT=ON -DBUILD_MODULE_PYBIND11=ON -DPYTHON_EXECUTABLE=/usr/bin/python .. && \
    make -j $ncores && \
    if [ "x$run_tests" = "x" ]; then echo "Skipping tests..."; else make check; fi && \
    make package && \
    cmake -DPYTHON_EXECUTABLE=/usr/bin/python3 .. && \
    make -j $ncores && \
    make package && \
    make repackage && \
    ls -1 *.deb | grep -iv 'unspecified' | xargs sudo dpkg -i'
