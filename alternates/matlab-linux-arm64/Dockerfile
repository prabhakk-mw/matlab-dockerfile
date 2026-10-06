# Copyright 2026 The MathWorks, Inc.
# This Dockerfile builds a Docker® image containing the Linux® ARM64 build of MATLAB®.
# It unpacks a MATLAB tar.gz archive onto a configurable base image, installs the
# matlab-proxy Python package using pipx, and starts matlab-proxy-app by default so you
# can access the MATLAB desktop from a browser.
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
# Run it (matlab-proxy serves the MATLAB desktop on port 8888):
#
#   docker run --init -it -p 8888:8888 matlab:r2026b-ubuntu26.04-arm64

# Select the base image for the container. Use an apt-based image such as ubuntu:26.04
# or a dnf/yum-based image such as registry.access.redhat.com/ubi9/ubi. Any base is
# supported as long as a matching base-dependencies-arm64.txt file exists in
# https://github.com/mathworks-ref-arch/container-images/tree/main/matlab-deps/<release>/
ARG BASE_IMAGE=ubuntu:26.04

FROM ${BASE_IMAGE}

# Specify which MATLAB release the archive contains. Use uppercase, for example R2026b.
ARG MATLAB_RELEASE=R2026b

# Specify the location in which to unpack MATLAB.
ARG MATLAB_INSTALL_LOCATION="/opt/matlab/${MATLAB_RELEASE}"

# Specify the name of the MATLAB ARM64 archive to unpack. The archive must be present in
# the matlab-arm-image folder next to this Dockerfile.
ARG MATLAB_TGZ=MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz

# Specify which base-dependencies-arm64.txt to use. This must match the folder name of
# your base image in the container-images repository (for example ubuntu26.04 or ubi9).
ARG MATLAB_DEPS_OS=ubuntu26.04

# Specify license server information using the format: port@hostname
ARG LICENSE_SERVER

# Specify a custom pip package index URL, for example a private PyPI mirror. Leave this
# unset to install matlab-proxy from the public Python Package Index.
ARG PIP_INDEX_URL

# Install the MATLAB operating-system dependencies, together with the tools required to
# fetch the dependency list and to install matlab-proxy. The dependency list is downloaded
# at build time from the container-images repository so that it always matches the release.
# Xvfb and a window manager are also installed because matlab-proxy needs a virtual display
# and a window manager (used for Simulink Online) to run the MATLAB desktop headlessly;
# neither is part of the MATLAB dependency list. The window manager is fluxbox on
# Debian/Ubuntu. fluxbox is not packaged for RHEL-family arm64, so openbox (the closest
# readily available equivalent) is installed there instead.
# This step supports both apt-based (Debian/Ubuntu) and dnf/yum-based (RHEL/UBI) images.
RUN set -e; \
    MATLAB_RELEASE_LC="$(echo "${MATLAB_RELEASE}" | tr '[:upper:]' '[:lower:]')"; \
    DEPS_URL="https://raw.githubusercontent.com/mathworks-ref-arch/container-images/main/matlab-deps/${MATLAB_RELEASE_LC}/${MATLAB_DEPS_OS}/base-dependencies-arm64.txt"; \
    if command -v apt-get >/dev/null 2>&1; then \
        export DEBIAN_FRONTEND=noninteractive; \
        apt-get update; \
        apt-get install --no-install-recommends --yes \
            ca-certificates wget sudo xvfb fluxbox python3 python3-pip python3-venv pipx; \
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
# available on the PATH for every user in the container. When PIP_INDEX_URL is set, pip and
# pipx install from that package index instead of the public Python Package Index.
ENV PIPX_HOME=/opt/pipx PIPX_BIN_DIR=/usr/local/bin
RUN if [ -n "${PIP_INDEX_URL}" ]; then export PIP_INDEX_URL="${PIP_INDEX_URL}"; fi \
    && pipx install matlab-proxy

# Add a non-root "matlab" user and grant it passwordless sudo. useradd is used because it
# is available on both Debian/Ubuntu and RHEL/UBI base images.
RUN useradd --create-home --shell /bin/bash matlab \
    && echo "matlab ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/matlab \
    && chmod 0440 /etc/sudoers.d/matlab

# Note: Uncomment one of the following two ways to configure the license server.

# Option 1. Specify the host and port of the machine that serves the network licenses
# if you want to store the license information in an environment variable. This
# is the preferred option. You can either use a build variable, like this:
# --build-arg LICENSE_SERVER=27000@MyServerName or you can specify the license server
# directly using: ENV MLM_LICENSE_FILE=27000@flexlm-server-name
ENV MLM_LICENSE_FILE=$LICENSE_SERVER

# Option 2. Alternatively, you can put a license file into the container.
# Enter the details of the license server in this file and uncomment the following line.
# COPY network.lic ${MATLAB_INSTALL_LOCATION}/licenses/

# The following environment variables allow MathWorks to understand how this MathWorks
# product (MATLAB Dockerfile) is being used. This information helps us make MATLAB even better.
# Your content, and information about the content within your files, is not shared with MathWorks.
# To opt out of this service, delete the environment variables defined in the following line.
# To learn more, see the Help Make MATLAB Even Better section in the accompanying README:
# https://github.com/mathworks-ref-arch/matlab-dockerfile#help-make-matlab-even-better
ENV MW_DDUX_FORCE_ENABLE=true MW_CONTEXT_TAGS=MATLAB:ARM64:MATLAB_PROXY:DOCKERFILE:V1

# Unpack the MATLAB archive into the install location. The archive is bind-mounted rather
# than copied so that the multi-gigabyte file never becomes a layer in the final image.
# The archive expands to a complete MATLAB root (bin/, appdata/, ...), so it is extracted
# directly into MATLAB_INSTALL_LOCATION. Finally, link matlab onto the PATH so that
# matlab-proxy can locate it.
#
# This is intentionally the last build step. Because the bind-mounted archive is part of
# this step's cache key, replacing the archive invalidates only this step; the operating-
# system dependency and matlab-proxy layers above are reused from the build cache.
RUN --mount=type=bind,source=matlab-arm-image/${MATLAB_TGZ},target=/tmp/matlab.tar.gz \
    mkdir -p "${MATLAB_INSTALL_LOCATION}" \
    && tar -xf /tmp/matlab.tar.gz -C "${MATLAB_INSTALL_LOCATION}" \
    && ln -s "${MATLAB_INSTALL_LOCATION}/bin/matlab" /usr/local/bin/matlab

# Set the user and work directory, expose the matlab-proxy port, and start matlab-proxy-app.
USER matlab
WORKDIR /home/matlab
EXPOSE 8888
ENV MWI_ENABLE_TOKEN_AUTH=false
ENV MWI_BASE_URL=/
ENV MWI_APP_PORT=8888
ENTRYPOINT ["matlab-proxy-app"]
