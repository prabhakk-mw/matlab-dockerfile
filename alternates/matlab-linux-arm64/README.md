# Run MATLAB on Linux ARM64 in a Container

The [`Dockerfile`](Dockerfile) in this subfolder builds a Docker&reg; image containing the Linux&reg; ARM64 build of MATLAB&reg;. Instead of installing MATLAB with the MATLAB Package Manager (*mpm*) or the installer, this solution unpacks a MATLAB `tar.gz` archive onto a base image that you choose. By default, the image starts MATLAB in an interactive command-line prompt. The image also installs the [matlab-proxy](https://github.com/mathworks/matlab-proxy) Python&reg; package using [pipx](https://pipx.pypa.io/), so you can override the entrypoint to access the MATLAB desktop from a web browser.

Use this solution to run MATLAB on a machine with an ARM64 (`aarch64`) processor. The image runs as a Linux container, so the host operating system can be any platform whose Docker engine runs Linux ARM64 containers, including Linux, Windows on ARM, and Apple silicon Macs. On x86-64 hardware, use the [`Dockerfile`](../../Dockerfile) in the top-level repository instead.

## Requirements

* A machine with an ARM64 (`aarch64`) processor and a Docker engine that runs Linux ARM64 containers. The host operating system can be Linux, Windows on ARM, or an Apple silicon Mac; on Windows and macOS, use Docker Desktop, which runs Linux containers in a lightweight virtual machine.
* A Docker environment with [BuildKit enabled](https://docs.docker.com/build/buildkit/#getting-started) (default in current Docker releases). The build uses a bind mount so that the large MATLAB archive never becomes a layer in the final image.
* A working internet connection at build time. The build downloads the MATLAB operating-system dependency list for your base image from the [container-images](https://github.com/mathworks-ref-arch/container-images) repository and installs matlab-proxy from the Python Package Index.
* The MATLAB ARM64 archive for this offering is already licensed, so you do not need to configure a license to run MATLAB.

## Get the Files

Clone this repository and navigate to this subfolder.

```bash
git clone https://github.com/mathworks-ref-arch/matlab-dockerfile.git
cd matlab-dockerfile/alternates/matlab-linux-arm64
```

## Download the MATLAB ARM64 Archive

Download the Linux ARM64 build of MATLAB and place the `tar.gz` archive in the [`matlab-arm-image`](matlab-arm-image) folder next to the Dockerfile. For instructions, see [`matlab-arm-image/README.md`](matlab-arm-image/README.md). Do not extract the archive.

After this step, the folder contains the archive, for example:

```
matlab-arm-image/
├── README.md
└── MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz
```

If your archive has a different filename, pass it to the build with the `MATLAB_TGZ` build argument.

## Build Instructions

This folder is the Docker build context, so run `docker build` from here.

The examples tag images as `matlab:<release>-<base>-<arch>`, for example `matlab:r2026b-ubuntu26.04-arm64`. Encoding the base image and architecture in the tag keeps variants that are built from the same release distinguishable. Use the OCI architecture name `arm64` (not `aarch64`). Because the MATLAB archive is currently arm64 only, these images are tagged per architecture rather than published as a multi-architecture manifest list.

### Build on an apt-based image (default)

By default, the Dockerfile builds on `ubuntu:26.04`.

```bash
docker build -t matlab:r2026b-ubuntu26.04-arm64 .
```

### Build on a dnf/yum-based image (RHEL family)

To build on a Red Hat&reg;-compatible image, set the base image and the matching dependency folder. RHEL 9&ndash;compatible distributions reuse the `ubi9` dependency list.

For example, to build on Rocky Linux&reg;:

```bash
docker build \
  --build-arg BASE_IMAGE=rockylinux/rockylinux:9 \
  --build-arg MATLAB_DEPS_OS=ubi9 \
  -t matlab:r2026b-rockylinux9-arm64 .
```

The Dockerfile detects whether the base image uses `apt-get` or `dnf`/`yum` and installs the dependencies accordingly. On RHEL-family images, it enables the CodeReady Builder (CRB) and EPEL repositories to obtain `Xvfb` and the window manager, and installs `python3.12` because the default `python3` (3.9) is too old for matlab-proxy.

> **Note**
> **Window manager:** matlab-proxy uses the `openbox` window manager, which is required only for Simulink&reg; Online. The Dockerfile installs `openbox` on all base images. Standard MATLAB desktop graphics (plots and figures) work through `Xvfb`.
>
> **Red Hat UBI:** The `Xvfb` and `openbox` packages are not available in the default UBI repositories, so the dnf build is verified against Rocky Linux. To build on UBI, use a base image or repository configuration that provides these packages.

## Run Instructions

The image uses `matlab` as its entrypoint, so arguments you pass to `docker run` are passed to MATLAB. Use the `--init` flag so that the container stops gracefully when a `docker stop` or `docker kill` command is issued.

The MATLAB ARM64 archive for this offering is already licensed, so you do not need to configure a license to run the container.

### Run MATLAB in an Interactive Command Prompt

By default, the container starts MATLAB in an interactive command-line prompt.

```bash
docker run --init -it --rm matlab:r2026b-ubuntu26.04-arm64
```

### Run MATLAB in Batch Mode

To start the container, run a MATLAB command, and then exit, pass `-batch` and the command to `docker run`. For example, to run `ver`, which displays the version of MATLAB and other installed products:

```bash
docker run --init --rm matlab:r2026b-ubuntu26.04-arm64 -batch ver
```

### Run the MATLAB Desktop in a Browser with matlab-proxy

The image installs [matlab-proxy](https://github.com/mathworks/matlab-proxy), which serves the MATLAB desktop over HTTP. To use it, override the entrypoint with `matlab-proxy-app` and publish the matlab-proxy port.

```bash
docker run --init -it --rm -p 8888:8888 \
  --entrypoint matlab-proxy-app matlab:r2026b-ubuntu26.04-arm64
```

Open the URL that matlab-proxy prints in the terminal (by default `http://localhost:8888`) in a browser to access the MATLAB desktop.

## Customize the Image

The [Dockerfile](Dockerfile) supports the following Docker build-time variables.

| Argument Name | Default value | Effect |
|---|---|---|
| `BASE_IMAGE` | `ubuntu:26.04` | The base image for the container. Use any apt-based or dnf/yum-based image that has a matching `base-dependencies-arm64.txt` file in the [container-images](https://github.com/mathworks-ref-arch/container-images/tree/main/matlab-deps) repository. |
| `MATLAB_DEPS_OS` | `ubuntu26.04` | The folder name in the container-images repository that contains `base-dependencies-arm64.txt` for your base image, for example `ubuntu26.04` or `ubi9`. This must correspond to `BASE_IMAGE`. |
| `MATLAB_RELEASE` | `R2026b` | The MATLAB release that the archive contains, in uppercase. For example: `R2026b`. |
| `MATLAB_TGZ` | `MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz` | The filename of the MATLAB ARM64 archive in the `matlab-arm-image` folder. |
| `MATLAB_INSTALL_LOCATION` | `/opt/matlab/R2026b` | The path at which to unpack MATLAB. |
| `PIP_INDEX_URL` | *unset* | A custom pip package index URL, for example a private PyPI mirror. Leave unset to install matlab-proxy from the public Python Package Index. |

Use these arguments with the `docker build` command to customize your image. Alternatively, you can change the default values for these arguments directly in the [Dockerfile](Dockerfile).

### Build an Image with MATLAB Unpacked to a Specific Location

```bash
docker build --build-arg MATLAB_INSTALL_LOCATION=/opt/matlab -t matlab:r2026b-ubuntu26.04-arm64 .
```

## More MATLAB Docker Resources
For more MATLAB Docker resources, see [More MATLAB Docker Resources](../../README.md#more-matlab-docker-resources).

## Help Make MATLAB Even Better
You can help improve MATLAB by providing user experience information on how you use MathWorks products. Your participation ensures that you are represented and helps us design better products. To opt out of this service, delete the following line in the Dockerfile:
```Dockerfile
ENV MW_DDUX_FORCE_ENABLE=true MW_CONTEXT_TAGS=MATLAB:ARM64:DOCKERFILE:V1
```

To learn more, see the documentation: [Help Make MATLAB Even Better - Frequently Asked Questions](https://www.mathworks.com/support/faq/user_experience_information_faq.html).

## Feedback
We encourage you to try this repository with your environment and provide feedback. If you encounter a technical issue or have an enhancement request, create an issue [here](https://github.com/mathworks-ref-arch/matlab-dockerfile/issues).

---
Copyright 2026 The MathWorks, Inc.
