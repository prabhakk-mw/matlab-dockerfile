# Copyright 2026 The MathWorks, Inc.
# This Dockerfile builds a Docker® image containing the Linux® ARM64 build of MATLAB®.
# It unpacks a MATLAB tar.gz archive onto a configurable base image and starts MATLAB in
# a command-line prompt by default. The image also installs the matlab-proxy Python
# package using pipx, so you can override the entrypoint to access the MATLAB desktop from
# a browser.
#
# Before you build, download the Linux ARM64 MATLAB archive and place it in the
# matlab-arm-image folder next to this Dockerfile. See matlab-arm-image/README.md for
# instructions.
#
# Build from this directory (this folder is the build context):
#
#   # Ubuntu (default)
#   docker build -t matlab:r2026b-ubuntu26.04-arm64 .
#
#   # Red Hat UBI9
#   docker build \
#     --build-arg BASE_IMAGE=registry.access.redhat.com/ubi9/ubi \
#     --build-arg MATLAB_DEPS_OS=ubi9 \
#     -t matlab:r2026b-ubi9-arm64 .
#
# Run MATLAB in an interactive command-line prompt (the default):
#
#   docker run --init -it --rm matlab:r2026b-ubuntu26.04-arm64
#
# Run a MATLAB command in batch mode and exit:
#
#   docker run --init --rm matlab:r2026b-ubuntu26.04-arm64 -batch ver
#
# Serve the MATLAB desktop in a browser on port 8888 with matlab-proxy:
#
#   docker run --init -it --rm -p 8888:8888 \
#     --entrypoint matlab-proxy-app matlab:r2026b-ubuntu26.04-arm64

# Select the base image for the container. (default : ubuntu:26.04)
# Use an apt-based image such as ubuntu:26.04
# or a dnf/yum-based image such as registry.access.redhat.com/ubi9/ubi. Any base is
# supported as long as a corresponding base-dependencies-arm64.txt file from
# https://github.com/mathworks-ref-arch/container-images/tree/main/matlab-deps/<release>/ 
# can be used to install the MATLAB dependencies. 
ARG BASE_IMAGE=ubuntu:26.04

# Specify which base-dependencies-arm64.txt to use. This must match the folder name of
# your base image in the container-images repository (for example ubuntu26.04 or ubi9).
ARG MATLAB_DEPS_OS=ubuntu26.04


FROM ${BASE_IMAGE}

# Specify which MATLAB release the archive contains.
ARG MATLAB_RELEASE=R2026b

# Specify the name of the MATLAB ARM64 archive to unpack. The archive must be present in
# the matlab-arm-image folder next to this Dockerfile.
ARG MATLAB_TGZ=MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz

# (Optional) Specify the location in which to unpack MATLAB.
ARG MATLAB_INSTALL_LOCATION="/opt/matlab/${MATLAB_RELEASE}"

# (Optional) Specify a custom pip package index URL, for example a private PyPI mirror.
ARG PIP_INDEX_URL


### Installing MATLAB Dependencies and matlab-proxy
RUN set -e; \
    MATLAB_RELEASE_LC="$(echo "${MATLAB_RELEASE}" | tr '[:upper:]' '[:lower:]')"; \
    DEPS_URL="https://raw.githubusercontent.com/mathworks-ref-arch/container-images/main/matlab-deps/${MATLAB_RELEASE_LC}/${MATLAB_DEPS_OS}/base-dependencies-arm64.txt"; \
    if command -v apt-get >/dev/null 2>&1; then \
        export DEBIAN_FRONTEND=noninteractive; \
        apt-get update; \
        apt-get install --no-install-recommends --yes \
            ca-certificates wget sudo xvfb openbox python3 python3-pip python3-venv pipx; \
        wget -qO /tmp/base-dependencies.txt "${DEPS_URL}"; \
        apt-get install --no-install-recommends --yes $(cat /tmp/base-dependencies.txt); \
        apt-get clean; \
        apt-get --yes autoremove; \
        rm -rf /var/lib/apt/lists/* /tmp/base-dependencies.txt; \
    elif command -v dnf >/dev/null 2>&1 || command -v yum >/dev/null 2>&1; then \
        PKG="$(command -v dnf || command -v yum)"; \
        # Enable the repositories that provide Xvfb (CodeReady Builder / CRB) and openbox
        # (EPEL). These are available on Rocky Linux, AlmaLinux, and CentOS Stream. On Red
        # Hat UBI, these packages are not in the default repositories; see the README.
        "${PKG}" install --disableplugin=subscription-manager --assumeyes dnf-plugins-core || true; \
        "${PKG}" config-manager --set-enabled crb 2>/dev/null \
            || "${PKG}" config-manager --set-enabled powertools 2>/dev/null || true; \
        "${PKG}" install --disableplugin=subscription-manager --assumeyes epel-release || true; \
        # RHEL 9 ships Python 3.9 as the default python3, which is too old for current
        # pipx and matlab-proxy. Install python3.12 from AppStream and use it for pipx.
        "${PKG}" install --disableplugin=subscription-manager --assumeyes \
            ca-certificates wget sudo xorg-x11-server-Xvfb openbox python3.12 python3.12-pip; \
        wget -qO /tmp/base-dependencies.txt "${DEPS_URL}"; \
        "${PKG}" install --disableplugin=subscription-manager --assumeyes $(cat /tmp/base-dependencies.txt); \
        python3.12 -m pip install --no-cache-dir pipx; \
        "${PKG}" clean all; \
        rm -rf /tmp/base-dependencies.txt; \
    else \
        echo "No supported package manager (apt-get, dnf, or yum) was found in ${BASE_IMAGE}." >&2; \
        exit 1; \
    fi

# Install matlab-proxy with pipx into a shared location so that matlab-proxy-app is
# available on the PATH for every user in the container. matlab-proxy is not used by the
# default command-line entrypoint, but it lets you serve the MATLAB desktop in a browser
# by overriding the entrypoint with matlab-proxy-app (see the README). When PIP_INDEX_URL
# is set, pip and pipx install from that package index instead of the public Python
# Package Index.
ENV PIPX_HOME=/opt/pipx PIPX_BIN_DIR=/usr/local/bin
RUN if [ -n "${PIP_INDEX_URL}" ]; then export PIP_INDEX_URL="${PIP_INDEX_URL}"; fi \
    && pipx install matlab-proxy

# Add a non-root "matlab" user and grant it passwordless sudo. 
RUN useradd --create-home --shell /bin/bash matlab \
    && echo "matlab ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/matlab \
    && chmod 0440 /etc/sudoers.d/matlab


# Unpack the MATLAB archive into the install location. The archive is bind-mounted rather
# than copied so that the multi-gigabyte file never becomes a layer in the final image.
# The archive expands to a complete MATLAB root (bin/, appdata/, ...), so it is extracted
# directly into MATLAB_INSTALL_LOCATION. 
RUN --mount=type=bind,source=matlab-arm-image/${MATLAB_TGZ},target=/tmp/matlab.tar.gz \
    mkdir -p "${MATLAB_INSTALL_LOCATION}" \
    && tar -xf /tmp/matlab.tar.gz -C "${MATLAB_INSTALL_LOCATION}" \
    && ln -s "${MATLAB_INSTALL_LOCATION}/bin/matlab" /usr/local/bin/matlab

# Switch to local user
USER matlab
WORKDIR /home/matlab

# configure entrypoint
ENV MW_DDUX_FORCE_ENABLE=true MW_CONTEXT_TAGS=MATLAB:ARM64:DOCKERFILE:V1

# The following environment variables configure matlab-proxy when you serve the MATLAB
# desktop by overriding the entrypoint with matlab-proxy-app (see the README). They have
# no effect on the default command-line MATLAB entrypoint.
ENV MWI_ENABLE_TOKEN_AUTH=false
ENV MWI_BASE_URL=/
ENV MWI_APP_PORT=8888
ENV MWI_USE_EXISTING_LICENSE=True
EXPOSE 8888

# Start MATLAB in a command-line prompt by default. Pass arguments such as "-batch ver"
# to the "docker run" command to run them in MATLAB. To serve the MATLAB desktop in a
# browser instead, override the entrypoint with matlab-proxy-app (see the README).
ENTRYPOINT ["matlab"]
CMD [""]
