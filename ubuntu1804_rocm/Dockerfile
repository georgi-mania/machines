FROM rocm/dev-ubuntu-18.04:4.2

LABEL description="Ubuntu 18.04 with Acts and HIP dependencies"
LABEL maintainer="Paul Gessinger <paul.gessinger@cern.ch"
# increase whenever any of the RUN commands change
LABEL version="1"

# DEBIAN_FRONTEND ensures non-blocking operation (tzdata is a problem)
ENV DEBIAN_FRONTEND noninteractive

# Add the ROCm/HIP binaries to the path.
ENV PATH /opt/rocm/bin:${PATH}

# In their infinite wisdom, AMD have modified their apt repo signing key.
ADD rocm.gpg.key .
RUN apt-key add rocm.gpg.key

# install dependencies from the package manager.
#
# see also https://root.cern.ch/build-prerequisites
RUN apt-get update -y \
  && apt-get install -y \
    build-essential \
    libubsan1 libasan5 libgcc-8-dev gcc-8-base gcc-8 cpp-8 g++-8 \
    curl \
    git \
    freeglut3-dev \
    libexpat-dev \
    libftgl-dev \
    libgl2ps-dev \
    libglew-dev \
    libgsl-dev \
    liblz4-dev \
    liblzma-dev \
    libpcre3-dev \
    libtbb-dev \
    libx11-dev \
    libxext-dev \
    libxft-dev \
    libxpm-dev \
    libxerces-c-dev \
    libzstd-dev \
    ninja-build \
    python3 \
    python3-dev \
    python3-pip \
    rsync \
    zlib1g-dev \
    ccache \
  && apt-get clean -y

# manual builds for hep-specific packages
ENV GET curl --location --silent --create-dirs
ENV UNPACK_TO_SRC tar -xz --strip-components=1 --directory src
ENV PREFIX /usr/local

# CMake 3.17.3 version in APT is too old
# requires rsync; installation uses `rsync` instead of `install`
RUN mkdir src \
  && ${GET} https://github.com/Kitware/CMake/releases/download/v3.17.3/cmake-3.17.3-Linux-x86_64.tar.gz \
    | ${UNPACK_TO_SRC} \
  && rsync -ruv src/ ${PREFIX} \
  && cd .. \
  && rm -rf src

# Eigen 3.3.7, has CUDA fix that we need
RUN mkdir src \
  && ${GET} https://gitlab.com/libeigen/eigen/-/archive/3.3.7/eigen-3.3.7.tar.gz \
    | ${UNPACK_TO_SRC} \
  && cmake -B build -S src -GNinja \
    -DCMAKE_INSTALL_PREFIX=${PREFIX} \
  && cmake --build build -- install \
  && rm -rf build src

# Boost 1.71.0, version in APT is too old
RUN mkdir src \
  && ${GET} https://boostorg.jfrog.io/artifactory/main/release/1.71.0/source/boost_1_71_0.tar.gz \
    | ${UNPACK_TO_SRC} \
  && cd src \
  && echo "using gcc : 8 : /usr/bin/g++-8 ;" > tools/build/src/user-config.jam \
  && echo "using python : 3.6 : /usr/bin/python3 : /usr/include/python3.6 : /usr/lib ;" >> tools/build/src/user-config.jam \
  && ./bootstrap.sh --prefix=${PREFIX} variant=release link=shared cxxflags="-std=c++17" \
  && ./b2 install variant=release link=shared cxxflags="-std=c++17" \
  && cd .. \
  && rm -rf src

# Geant4
RUN mkdir src \
  && ${GET} https://geant4-data.web.cern.ch/geant4-data/releases/geant4.10.06.p01.tar.gz \
    | ${UNPACK_TO_SRC} \
  && cmake -B build -S src -GNinja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=${PREFIX} \
    -DGEANT4_BUILD_CXXSTD=17 \
    -DGEANT4_INSTALL_DATA=OFF \
    -DGEANT4_USE_GDML=ON \
    -DGEANT4_USE_SYSTEM_EXPAT=ON \
    -DGEANT4_USE_SYSTEM_ZLIB=ON \
  && cmake --build build -- install \
  && rm -rf build src

# HepMC3
RUN mkdir src \
  && ${GET} https://hepmc.web.cern.ch/hepmc/releases/HepMC3-3.2.1.tar.gz \
    | ${UNPACK_TO_SRC} \
  && cmake -B build -S src -GNinja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=${PREFIX} \
    -DHEPMC3_BUILD_STATIC_LIBS=OFF \
    -DHEPMC3_ENABLE_PYTHON=OFF \
    -DHEPMC3_ENABLE_ROOTIO=OFF \
    -DHEPMC3_ENABLE_SEARCH=OFF \
  && cmake --build build -- install \
  && rm -rf build src

# Pythia8
# requires rsync; installation uses `rsync` instead of `install`
RUN mkdir src \
  && ${GET} https://pythia.org/download/pythia82/pythia8244.tgz \
    | ${UNPACK_TO_SRC} \
  && cd src \
  && ./configure --enable-shared --prefix=${PREFIX} \
  && make install \
  && cd .. \
  && rm -rf src

# xxHash
RUN mkdir src \
  && ${GET} https://github.com/Cyan4973/xxHash/archive/v0.7.3.tar.gz \
    | ${UNPACK_TO_SRC} \
  && cmake -B build -S src/cmake_unofficial -GNinja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=${PREFIX} \
  && cmake --build build -- install \
  && rm -rf build src


# ROOT
RUN mkdir src \
  && ${GET} https://root.cern/download/root_v6.20.04.source.tar.gz \
    | ${UNPACK_TO_SRC} \
  && cmake -B build -S src -GNinja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CXX_STANDARD=17 \
    -DCMAKE_INSTALL_PREFIX=${PREFIX} \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    -Dfail-on-missing=ON \
    -Dgminimal=ON \
    -Dgdml=ON \
    -Dopengl=ON \
    -Dpyroot=ON \
  && cmake --build build -- install \
  && rm -rf build src

# DD4hep
# requires Geant4 and ROOT and must come last
#
# environment variables needed for DD4hep to find ROOT libraries
ENV LD_LIBRARY_PATH /usr/local/lib
ENV PYTHON_PATH /usr/local/lib
RUN mkdir src \
  && ${GET} https://github.com/AIDASoft/DD4hep/archive/v01-11-02.tar.gz \
    | ${UNPACK_TO_SRC} \
  && cmake -B build -S src -GNinja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CXX_STANDARD=17 \
    -DCMAKE_INSTALL_PREFIX=${PREFIX} \
    -DCMAKE_PREFIX_PATH=${PREFIX} \
    -DBUILD_TESTING=OFF \
    -DDD4HEP_BUILD_PACKAGES="DDG4" \
    -DDD4HEP_IGNORE_GEANT4_TLS=ON \
    -DDD4HEP_USE_GEANT4=ON \
    -DDD4HEP_USE_XERCESC=ON \
  && cmake --build build -- install \
  && rm -rf build src
