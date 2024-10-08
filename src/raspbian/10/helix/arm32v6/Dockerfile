ARG ROOTFS_DIR=/crossrootfs

FROM mcr.microsoft.com/dotnet-buildtools/prereqs:raspbian-10-crossdeps as builder

ARG ROOTFS_DIR

# Obtain arcade scripts used to build rootfs
RUN git config --global user.email builder@dotnet-buildtools-prereqs-docker && \
    git clone --depth 1 --single-branch https://github.com/dotnet/arcade /scripts

# Build the rootfs
RUN /scripts/eng/common/cross/build-rootfs.sh armv6 raspbian --skipunmount

FROM scratch
ARG ROOTFS_DIR

# There doesn't seem to be an official Raspberry Pi OS Docker image
# But we can use a crossrootfs from our image generation scripts, and
# extract into the image root

COPY --from=builder $ROOTFS_DIR /

ENV _PYTHON_HOST_PLATFORM=linux_armv6l

# Install Helix Dependencies

RUN apt-get update && \
    apt-get install -y \
        autoconf \
        automake \
        at \
        build-essential \
        ca-certificates \
        curl \
        gcc \
        gdb \
        git \
        iputils-ping \
        libcurl4 \
        libffi-dev \
        libgdiplus \
        libicu-dev \
        libssl-dev \
        libtool \
        libunwind8 \
        locales \
        locales-all \
        python3-dev \
        python3-pip \
        sudo \
        tzdata \
        unzip \
    && rm -rf /var/lib/apt/lists/* \
    && c_rehash \
    \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

ENV LANG=en_US.utf8

RUN ln -sf /usr/bin/python3 /usr/bin/python && \
    pip download --no-deps helix-scripts --index-url https://dnceng.pkgs.visualstudio.com/public/_packaging/helix-client-prod/pypi/simple && \
    export CRYPTOGRAPHY_DONT_BUILD_RUST=1 && \
    pip install ./helix_scripts-*-py3-none-any.whl

# Create helixbot user and give rights to sudo without password
# (we use two users here to ensure volume mounting works with two possible UIDs of the host UID)
# additionally, preinstall the virtualenv packages used for VSTS reporting to save time
RUN /usr/sbin/adduser --disabled-password --gecos '' --uid 1001 --shell /bin/bash --ingroup adm helixbot && \
    /usr/sbin/adduser --disabled-password --gecos '' --uid 1000 --shell /bin/bash --ingroup adm helixbot2 && \
    chmod 755 /root && \
    echo "helixbot ALL=(ALL)       NOPASSWD: ALL" > /etc/sudoers && \
    echo "helixbot2 ALL=(ALL)       NOPASSWD: ALL" > /etc/sudoers

USER helixbot

RUN python -m venv /home/helixbot/.vsts-env
