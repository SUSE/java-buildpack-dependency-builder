FROM cloudfoundry/cflinuxfs2

RUN apt-get update && apt-get install -y \
    apt-utils \
    autoconf \
    automake \
    curl \
    g++ \
    git \
    libssl-dev \
    libtool \
    make \
    openjdk-7-jdk \
    python \
    python-dev \
    python-pip \
    unzip \
    zlib1g-dev \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# curl
RUN git clone --depth=1 -b curl-7_55_1 https://github.com/curl/curl.git /tmp/curl && \
    cd /tmp/curl && \
    ./buildconf && \
    ./configure --disable-ldap --disable-shared --without-libssh2 \
                --without-librtmp --without-libidn --enable-static \
                --with-pic --with-ssl && \
    make -j && make install && \
    cd ~ && rm -rf /tmp/curl

# cmake
RUN git clone -b v3.6.2 https://github.com/Kitware/CMake.git /tmp/cmake && \
    cd /tmp/cmake && \
    ./bootstrap && \
    make && make install && \
    cd ~ && rm -rf /tmp/cmake

# gflags
RUN git clone --depth=1 -b v2.1.2 https://github.com/gflags/gflags.git /tmp/gflags && \
    cd /tmp/gflags && \
    mkdir build && cd build && \
    cmake -DCMAKE_CXX_FLAGS=-fpic -DGFLAGS_NAMESPACE=google .. && \
    make -j && make install && \
    cd ~ && rm -rf /tmp/gflags

# google-glog
RUN mkdir /tmp/glog && cd /tmp/glog && \
    curl -sL https://github.com/google/glog/archive/v0.3.5.tar.gz | \
        tar xzv --strip=1 && ./configure --with-pic && \
    make -j && make install && \
    cd ~ && rm -rf /tmp/glog

# openssl
# This openssl (compiled with -fPIC) is used to statically link into the agent
# shared library.  We still install libssl-dev above since getting libcurl to
# link with the custom version is tricky, see
# http://askubuntu.com/questions/475670/how-to-build-curl-with-the-latest-openssl.
# Also, intentionally not using -j as OpenSSL < 1.1.0 not quite supporting the
# parallel builds, see
# https://github.com/openssl/openssl/issues/5762#issuecomment-376622684.
RUN mkdir /tmp/openssl && cd /tmp/openssl && \
    curl -sL https://www.openssl.org/source/openssl-1.0.2o.tar.gz | \
        tar xzv --strip=1 && \
    ./config no-shared -fPIC --openssldir=/usr/local/ssl && \
    make && make install_sw && \
    cd ~ && rm -rf /tmp/openssl

# gRPC & protobuf
# Use the protobuf version from gRPC for everything to avoid conflicting
# versions to be linked in. Disable OpenSSL embedding: when it's on, the build
# process of gRPC puts the OpenSSL static object files into the gRPC archive
# which causes link errors later when the agent is linked with the static
# OpenSSL library itself.
RUN git clone --depth=1 --recursive -b v1.11.0 https://github.com/grpc/grpc.git /tmp/grpc && \
    cd /tmp/grpc && \
    cd third_party/protobuf && \
    ./autogen.sh && ./configure --with-pic CXXFLAGS=-Os && make && make install && ldconfig && \
    cd ../.. && \
    CPPFLAGS="-I /usr/local/ssl/include" make CONFIG=opt EMBED_OPENSSL=false V=1 HAS_SYSTEM_OPENSSL_NPN=0 && \
    CPPFLAGS="-I /usr/local/ssl/include" make CONFIG=opt EMBED_OPENSSL=false V=1 HAS_SYSTEM_OPENSSL_NPN=0 install && \
    rm -rf /tmp/grpc

RUN pip install awscli --ignore-installed six
