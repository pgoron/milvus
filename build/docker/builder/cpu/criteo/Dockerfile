FROM filer-docker-registry-build.prod.crto.in/criteo-base-8-python:latest

#
# Docker image to build milvus-io/milvus
# Heavily inspired by https://github.com/milvus-io/milvus/blob/v2.5.11/build/docker/builder/cpu/ubuntu22.04/Dockerfile
# NB: image relies on external packages not replicated internally (vcpkg & connan will look for dependencies
#     on internet when invoked by milvus build), reason why we don't use filer-container-files for now.
#

# IMPORTANT: these version numbers are dependant of milvus version being built, update them when needed
ARG CONAN_VERSION=1.64.1
ARG GO_VERSION=1.24.1
ARG RUST_VERSION=1.83
ARG VCPKG_VERSION=2025.04.09
ARG TARGETARCH=amd64


RUN dnf install -y wget curl cmake tar which \
    gcc-toolset-12 gcc-toolset-12-libatomic-devel ninja-build git make ccache openssl-devel zlib-devel zip unzip \
    lcov libtool m4 autoconf automake python3 python3-pip texinfo \
    pkg-config libuuid-devel libaio-devel openblas-devel \
    perl-IPC-Cmd perl-Digest-SHA && \
    ln -sf /usr/bin/python3.11 /usr/bin/python3 && \
    echo "source /opt/rh/gcc-toolset-12/enable" >> ~/.bashrc

# knowhere relies on openblas headers but doesn't explicitly look for them via cmake files
ENV CPLUS_INCLUDE_PATH=/usr/include/openblas

# use login shell to have .bashrc read at each invocation
SHELL ["bash", "-l", "-c"]

# install conan
RUN pip3 install conan==$CONAN_VERSION

# install vcpkg
RUN mkdir /opt/vcpkg &&  \
    wget -qO- vcpkg.tar.gz https://github.com/microsoft/vcpkg/archive/refs/tags/$VCPKG_VERSION.tar.gz | tar --strip-components=1 -xz -C /opt/vcpkg && \
    rm -rf vcpkg.tar.gz

ENV VCPKG_FORCE_SYSTEM_BINARIES=1
ENV VCPKG_DEFAULT_BINARY_CACHE=/home/milvus/.cache/vcpkg/archives

# install azure-identity-cpp azure-storage-blobs-cpp gtest via vcpkg
RUN mkdir -p $VCPKG_DEFAULT_BINARY_CACHE && \
    /opt/vcpkg/bootstrap-vcpkg.sh -disableMetrics && \
    ln -s /opt/vcpkg/vcpkg /usr/local/bin/vcpkg && \
    vcpkg version

RUN vcpkg install azure-identity-cpp azure-storage-blobs-cpp gtest  && \
    chmod -R 777 "/opt/vcpkg" && \
    chmod -R 777 $VCPKG_DEFAULT_BINARY_CACHE


# Install Go
ENV GOPATH=/go
ENV GOROOT=/usr/local/go
ENV GO111MODULE=on
ENV PATH=$GOPATH/bin:$GOROOT/bin:$PATH
RUN mkdir -p /usr/local/go && wget -qO- "https://go.dev/dl/go$GO_VERSION.linux-$TARGETARCH.tar.gz" | tar --strip-components=1 -xz -C /usr/local/go && \
    mkdir -p "$GOPATH/src" "$GOPATH/bin" "$GOPATH/pkg" && \
    go clean --modcache && \
    chmod -R 777 "$GOPATH" && chmod -R a+w $(go env GOTOOLDIR)

# refer: https://code.visualstudio.com/docs/remote/containers-advanced#_avoiding-extension-reinstalls-on-container-rebuild
RUN mkdir -p /home/milvus/.vscode-server/extensions \
    /home/milvus/.vscode-server-insiders/extensions

COPY --chown=0:0 build/docker/builder/entrypoint.sh /

# install rust
ENV CARGO_HOME=/home/milvus/.cargo
ENV RUSTUP_HOME=/home/milvus/.rustup
RUN curl https://sh.rustup.rs -sSf | \
    sh -s -- --default-toolchain=$RUST_VERSION -y

ENV PATH=$CARGO_HOME/bin:$PATH

RUN chmod -R 777 /home/milvus

ENTRYPOINT [ "/entrypoint.sh" ]
CMD ["tail", "-f", "/dev/null"]
