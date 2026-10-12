# Copyright 2026 The MathWorks, Inc.
# This Dockerfile builds a Docker® image containing the Linux® ARM64 build of MATLAB®.
# It builds on a prebuilt MATLAB dependencies (matlab-deps) image, which already has the
# MATLAB operating-system dependencies installed, so you do not have to pick a dependency
# list or download it at build time. The image starts MATLAB in a command-line prompt by
# default and also installs the matlab-proxy Python package using pipx, so you can override
# the entrypoint to access the MATLAB desktop from a browser.
#
# This Dockerfile supports two ways of providing MATLAB itself:
#
#   1. Embed MATLAB at build time. Place the MATLAB ARM64 tar.gz archive in the
#      matlab-arm-image folder next to this Dockerfile (see matlab-arm-image/README.md) and
#      build. The archive is unpacked into the image, which is then self-contained.
#
#   2. Mount MATLAB at run time. Build with no archive present to produce a smaller image
#      that does not embed MATLAB (for example, to publish as a prebuilt image), then mount
#      an extracted MATLAB root at MATLAB_INSTALL_LOCATION when you run the container.
#
# Choose a base image by setting BASE_IMAGE to the matlab-deps-arm64 tag that most closely
# matches your needs. The available tags are published here:
# https://github.com/mathworks-ref-arch/container-images/pkgs/container/container-images%2Fmatlab-deps-arm64
# For example, for R2026b: r2026b-ubuntu26.04 (default), r2026b-ubuntu24.04,
# r2026b-ubuntu22.04, r2026b-debian13, r2026b-debian12, r2026b-ubi9, or r2026b-ubi8.
#
# Build from this directory (this folder is the build context):
#
#   # Ubuntu 26.04 (default)
#   docker build -t matlab:r2026b-ubuntu26.04-arm64 .
#
#   # Red Hat UBI9
#   docker build \
#     --build-arg BASE_IMAGE=ghcr.io/mathworks-ref-arch/container-images/matlab-deps-arm64:r2026b-ubi9 \
#     -t matlab:r2026b-ubi9-arm64 .
#
# Run MATLAB in an interactive command-line prompt (the default). Add the -v mount only if
# MATLAB is not embedded in the image:
#
#   docker run --init -it --rm \
#     -v /path/to/R2026b:/opt/matlab/R2026b matlab:r2026b-ubuntu26.04-arm64
#
# Run a MATLAB command in batch mode and exit:
#
#   docker run --init --rm matlab:r2026b-ubuntu26.04-arm64 -batch ver
#
# Serve the MATLAB desktop in a browser on port 8888 with matlab-proxy:
#
#   docker run --init -it --rm -p 8888:8888 \
#     --entrypoint matlab-proxy-app matlab:r2026b-ubuntu26.04-arm64

# Select the prebuilt matlab-deps base image for the container. Pick the matlab-deps-arm64
# tag that most closely matches your needs. See the comment block above for the list of
# available tags.
ARG BASE_IMAGE=ghcr.io/mathworks-ref-arch/container-images/matlab-deps-arm64:r2026b-ubuntu26.04


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

# (Optional, RHEL family only) Set to "true" to add the repositories needed to install the
# matlab-proxy browser desktop dependencies (Xvfb and openbox) on base images that do not
# provide them, notably Red Hat UBI. When "true", the build adds EPEL (for openbox) and the
# CentOS Stream AppStream repository (for Xvfb). This mixes CentOS Stream packages into the
# image, which Red Hat does not support on UBI, so it is opt-in. It has no effect on
# apt-based (Ubuntu/Debian) images, where these packages are always installed. See the README.
ARG ADD_PROXY_DESKTOP_REPOS


# The prebuilt matlab-deps image already provides the MATLAB operating-system
# dependencies. This step adds the extra packages that matlab-proxy needs (Python and
# pipx), plus the Xvfb headless X server and the openbox window manager that the
# matlab-proxy browser desktop requires. The Dockerfile detects whether the base image
# uses apt-get (Ubuntu/Debian) or dnf/yum (RHEL family) and installs them accordingly.
#
# On apt-based images Xvfb and the fluxbox and openbox window managers are always
# installed. On RHEL-family images they come from AppStream/CRB (Xvfb) and EPEL (openbox);
# these are available on Rocky Linux,
# AlmaLinux, CentOS Stream, and subscribed RHEL, but not on Red Hat UBI. The install is
# best-effort, so a UBI image builds even when they are missing: command-line MATLAB and
# -batch still work, only the matlab-proxy browser desktop is unavailable. To get the
# desktop on UBI, build with --build-arg ADD_PROXY_DESKTOP_REPOS=true (see the README).
RUN set -e; \
    # Use the custom pip index for all pip/pipx invocations in this layer when provided.
    if [ -n "${PIP_INDEX_URL}" ]; then export PIP_INDEX_URL="${PIP_INDEX_URL}"; fi; \
    if command -v apt-get >/dev/null 2>&1; then \
        export DEBIAN_FRONTEND=noninteractive; \
        apt-get update; \
        apt-get install --no-install-recommends --yes \
            ca-certificates wget sudo xvfb openbox fluxbox python3 python3-pip python3-venv pipx; \
        apt-get clean; \
        apt-get --yes autoremove; \
        rm -rf /var/lib/apt/lists/*; \
    elif command -v dnf >/dev/null 2>&1 || command -v yum >/dev/null 2>&1; then \
        PKG="$(command -v dnf || command -v yum)"; \
        # Install the essential packages. RHEL 9 ships Python 3.9 as the default python3,
        # which is too old for current pipx and matlab-proxy, so install python3.12.
        "${PKG}" install --disableplugin=subscription-manager --assumeyes \
            ca-certificates wget sudo python3.12 python3.12-pip; \
        python3.12 -m pip install --no-cache-dir pipx; \
        # Enable the repositories that provide the matlab-proxy desktop dependencies: Xvfb
        # (in AppStream / CodeReady Builder) and openbox (in EPEL). On Rocky Linux, AlmaLinux,
        # CentOS Stream, and subscribed RHEL these are available, so this step suffices there.
        "${PKG}" install --disableplugin=subscription-manager --assumeyes dnf-plugins-core || true; \
        "${PKG}" config-manager --set-enabled crb 2>/dev/null \
            || "${PKG}" config-manager --set-enabled powertools 2>/dev/null || true; \
        "${PKG}" install --disableplugin=subscription-manager --assumeyes epel-release || true; \
        # Red Hat UBI does not ship Xvfb or the epel-release package. When ADD_PROXY_DESKTOP_REPOS
        # is "true", add EPEL from its RPM URL (for openbox) and the CentOS Stream AppStream
        # repository (for Xvfb). See the ARG comment and the README for the trade-offs.
        if [ "${ADD_PROXY_DESKTOP_REPOS}" = "true" ]; then \
            EL="$(rpm -E %rhel)"; \
            ARCH="$(rpm -E %{_arch})"; \
            CENTOS_KEY="https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Official-SHA256"; \
            "${PKG}" install --assumeyes "https://dl.fedoraproject.org/pub/epel/epel-release-latest-${EL}.noarch.rpm" || true; \
            rpm --import "${CENTOS_KEY}" || true; \
            { echo "[centos-stream-appstream]"; \
              echo "name=CentOS Stream ${EL} - AppStream"; \
              echo "baseurl=https://mirror.stream.centos.org/${EL}-stream/AppStream/${ARCH}/os/"; \
              echo "gpgcheck=1"; \
              echo "gpgkey=${CENTOS_KEY}"; \
              echo "enabled=1"; } > /etc/yum.repos.d/centos-stream-appstream.repo; \
        fi; \
        # Best-effort install so that bases without these packages (for example UBI built
        # without ADD_PROXY_DESKTOP_REPOS) still build. When Xvfb/openbox are missing, only
        # the matlab-proxy browser desktop is unavailable; command-line MATLAB and -batch work.
        "${PKG}" install --disableplugin=subscription-manager --assumeyes \
            xorg-x11-server-Xvfb openbox \
            || echo "WARNING: Xvfb and/or openbox could not be installed; the matlab-proxy browser desktop will not work on this image. Command-line MATLAB and -batch are unaffected."; \
        "${PKG}" clean all; \
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


# Set up the MATLAB install location and the "matlab" launcher symlink. If a MATLAB ARM64
# archive is present in the matlab-arm-image folder, unpack it into MATLAB_INSTALL_LOCATION
# so the image is self-contained. If no archive is present, the build still succeeds and
# produces an image that expects MATLAB to be mounted at MATLAB_INSTALL_LOCATION when you
# run the container (for example: docker run -v /path/to/R2026b:/opt/matlab/R2026b ...).
# This lets the same Dockerfile publish prebuilt images that do not embed MATLAB.
#
# The whole matlab-arm-image folder is bind-mounted (rather than a specific file) so that
# the build does not fail when the archive is absent, and so the multi-gigabyte archive
# never becomes a layer in the final image. The archive expands to a complete MATLAB root
# (bin/, appdata/, ...), so it is extracted directly into MATLAB_INSTALL_LOCATION. The
# matlab symlink is created unconditionally; it resolves once MATLAB is present at build
# time or mounted at run time.
RUN --mount=type=bind,source=matlab-arm-image,target=/tmp/matlab-arm-image \
    mkdir -p "${MATLAB_INSTALL_LOCATION}" \
    && if [ -f "/tmp/matlab-arm-image/${MATLAB_TGZ}" ]; then \
        echo "Unpacking ${MATLAB_TGZ} into ${MATLAB_INSTALL_LOCATION}"; \
        tar -xf "/tmp/matlab-arm-image/${MATLAB_TGZ}" -C "${MATLAB_INSTALL_LOCATION}"; \
    else \
        echo "No MATLAB archive found in matlab-arm-image; building an image without MATLAB embedded."; \
        echo "Mount MATLAB at ${MATLAB_INSTALL_LOCATION} when you run the container."; \
    fi \
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
