# Copyright 2026 The MathWorks, Inc.
#
# Builds a Docker® image containing the Linux® ARM64 build of MATLAB® on a prebuilt
# matlab-deps base image. MATLAB can be embedded at build time (place the tar.gz archive in
# the matlab-arm-image folder) or mounted at run time (build with no archive present). For
# build options, base images, and run instructions, see README.md and build/README.md.

ARG BASE_IMAGE=ghcr.io/mathworks-ref-arch/container-images/matlab-deps-arm64:r2026b-ubuntu26.04


FROM ${BASE_IMAGE}

ARG MATLAB_RELEASE=R2026b
# Name of the MATLAB ARM64 archive to unpack, expected in the matlab-arm-image folder.
ARG MATLAB_TGZ=MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz
ARG MATLAB_INSTALL_LOCATION="/opt/matlab/${MATLAB_RELEASE}"
ARG PIP_INDEX_URL


# Install the extra packages needed on top of matlab-deps, detecting the package manager.
# On Ubuntu/Debian also install matlab-proxy for the browser desktop; Red Hat UBI is
# command-line only, so it gets only sudo (see README).
RUN set -e; \
    if command -v apt-get >/dev/null 2>&1; then \
        export DEBIAN_FRONTEND=noninteractive; \
        apt-get update; \
        apt-get install --no-install-recommends --yes \
            ca-certificates wget sudo xvfb openbox fluxbox python3 python3-pip python3-venv pipx; \
        apt-get clean; \
        apt-get --yes autoremove; \
        rm -rf /var/lib/apt/lists/*; \
        export PIPX_HOME=/opt/pipx PIPX_BIN_DIR=/usr/local/bin; \
        if [ -n "${PIP_INDEX_URL}" ]; then export PIP_INDEX_URL="${PIP_INDEX_URL}"; fi; \
        pipx install matlab-proxy; \
    elif command -v dnf >/dev/null 2>&1 || command -v yum >/dev/null 2>&1; then \
        PKG="$(command -v dnf || command -v yum)"; \
        "${PKG}" install --disableplugin=subscription-manager --assumeyes sudo; \
        "${PKG}" clean all; \
    else \
        echo "No supported package manager (apt-get, dnf, or yum) was found in ${BASE_IMAGE}." >&2; \
        exit 1; \
    fi

# Add a non-root "matlab" user with passwordless sudo.
RUN useradd --create-home --shell /bin/bash matlab \
    && echo "matlab ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/matlab \
    && chmod 0440 /etc/sudoers.d/matlab


# Unpack MATLAB if the archive is present; otherwise build an image that expects MATLAB to be
# mounted at MATLAB_INSTALL_LOCATION at run time. The matlab-arm-image folder is bind-mounted
# (not COPYed) so the build still succeeds when the archive is absent and the multi-gigabyte
# archive never becomes an image layer. The symlink is created unconditionally; it resolves
# once MATLAB is present at build time or mounted at run time.
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

USER matlab
WORKDIR /home/matlab

# Opt out of user-experience data collection by removing this line (see README).
ENV MW_DDUX_FORCE_ENABLE=true MW_CONTEXT_TAGS=MATLAB:ARM64:DOCKERFILE:V1

# matlab-proxy configuration (Ubuntu/Debian only; inert on the command-line entrypoint and UBI).
ENV MWI_ENABLE_TOKEN_AUTH=false
ENV MWI_BASE_URL=/
ENV MWI_APP_PORT=8888
ENV MWI_USE_EXISTING_LICENSE=True
EXPOSE 8888

ENTRYPOINT ["matlab"]
CMD [""]
