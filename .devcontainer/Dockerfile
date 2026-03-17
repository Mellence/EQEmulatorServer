# This is a "dev" image for running eqemu in development, also for dev containers

FROM debian:12@sha256:bc960ef50e6feed90686c593361df158517556ed1d2d98e5d1df3724024e0f49
ARG USERNAME=peq
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN apt-get update \
    && apt-get install -y --no-install-recommends sudo \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m -s /bin/bash $USERNAME \
    && echo "$USERNAME ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

RUN mkdir -p /src && chown $USERNAME:$USERNAME /src


RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential libtool cmake curl debconf-utils git \
    ca-certificates libluabind-dev libsodium-dev liblua5.2-0 liblua5.2-dev \
    libmariadb-dev libssl-dev minizip make mariadb-client \
    locales nano zip unzip tar pkg-config autoconf automake m4 nasm uuid-dev iputils-ping wget \
    libcurl4-openssl-dev gdb libyaml-cpp-dev ccache ninja-build pv \
    libperl-dev libjson-perl libio-stringy-perl liblua5.1-dev \
    libboost-dev valgrind telnet libgoogle-perftools-dev google-perftools \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

USER $USERNAME
