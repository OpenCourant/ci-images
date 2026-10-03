# CI build + QA image for OpenCourant Linux x86_64 builds.
#
# Replaces the inaccessible Altair-internal image
# internal.artifacts.altair.com/radioss-docker-local/rd-linux64-or-rl8.10-bld-qa:ompi4.1.2-ts11
# with an equivalent environment rebuilt from public sources:
#   - Rocky Linux 8.10 (matches the "rl8.10" environment the QA scripts were tuned for)
#   - GCC/GFortran 11 via gcc-toolset-11 (documented requirement in HOWTO.md)
#   - OpenMPI 4.1.2 installed in /opt/openmpi (documented requirement in HOWTO.md)
#
# The inherited CI workflows run `source /root/.bashrc` before building, so the
# toolchain and MPI environment are exported there as well as via ENV.

FROM docker.io/rockylinux/rockylinux:8.10

SHELL ["/bin/bash", "-c"]

RUN dnf -y install dnf-plugins-core epel-release && \
    dnf config-manager --set-enabled powertools && \
    dnf -y update && \
    dnf -y install \
        gcc-toolset-11-toolchain \
        gcc-toolset-11-libasan-devel \
        gcc-toolset-11-libubsan-devel \
        cmake \
        make \
        perl \
        perl-Env \
        python3.11 \
        git \
        git-lfs \
        zip \
        unzip \
        tar \
        gzip \
        bzip2 \
        xz \
        wget \
        curl \
        which \
        diffutils \
        patch \
        file \
        findutils \
        hostname \
        procps-ng \
        iproute \
        glibc-static \
        libstdc++-static \
        zlib-devel \
        openssl-devel \
        ca-certificates && \
    dnf clean all && \
    rm -rf /var/cache/dnf

# Build scripts invoke `python3` and sometimes plain `python`.
# /usr/local/bin precedes /usr/bin on PATH, so these win over platform python 3.6.
RUN ln -s /usr/bin/python3.11 /usr/local/bin/python3 && \
    ln -s /usr/bin/python3.11 /usr/local/bin/python

# OpenMPI 4.1.2 built with gcc-toolset-11, installed in /opt/openmpi
# (preferred version per OpenCourant HOWTO.md)
ARG OPENMPI_VERSION=4.1.2
ARG OPENMPI_SHA256=a400719b04375cd704d2ed063a50e42d268497a3dfede342986ab7a8d7e8dcf0
RUN source /opt/rh/gcc-toolset-11/enable && \
    cd /tmp && \
    wget -q "https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-${OPENMPI_VERSION}.tar.gz" && \
    echo "${OPENMPI_SHA256}  openmpi-${OPENMPI_VERSION}.tar.gz" | sha256sum -c - && \
    tar xzf "openmpi-${OPENMPI_VERSION}.tar.gz" && \
    cd "openmpi-${OPENMPI_VERSION}" && \
    ./configure --prefix=/opt/openmpi > /tmp/ompi-configure.log && \
    make -j"$(nproc)" > /tmp/ompi-make.log && \
    make install > /tmp/ompi-install.log && \
    cd / && \
    rm -rf /tmp/openmpi-${OPENMPI_VERSION}* /tmp/ompi-*.log

# The CI workflows `source /root/.bashrc` before building
RUN echo 'source /opt/rh/gcc-toolset-11/enable' >> /root/.bashrc && \
    echo 'export PATH=/opt/openmpi/bin:$PATH' >> /root/.bashrc && \
    echo 'export LD_LIBRARY_PATH=/opt/openmpi/lib:$LD_LIBRARY_PATH' >> /root/.bashrc

ENV PATH=/opt/openmpi/bin:/opt/rh/gcc-toolset-11/root/usr/bin:/usr/local/bin:/usr/bin:/usr/sbin:/bin:/sbin \
    LD_LIBRARY_PATH=/opt/openmpi/lib

CMD ["/bin/bash"]
