# Build a MATLAB on Linux ARM64 Container Image

> **Most beta users do not need this page.** The simplest way to try this offering is to pull a prebuilt image and mount your MATLAB installation into it &mdash; see the [main README](../README.md). Use the instructions here only if you need or want to **build your own image**, for example to embed MATLAB in a self-contained image, to build on a different base operating system, or to publish images to your own registry.

The [`Dockerfile`](../Dockerfile) in this folder builds a Docker&reg; image containing the Linux&reg; ARM64 build of MATLAB&reg;. Instead of installing MATLAB with the MATLAB Package Manager (*mpm*) or the installer, it unpacks a MATLAB `tar.gz` archive onto a prebuilt [MATLAB dependencies (matlab-deps)](https://github.com/mathworks-ref-arch/container-images/pkgs/container/container-images%2Fmatlab-deps-arm64) image, so the MATLAB operating-system dependencies are already present and you just pick the prebuilt base image that most closely matches your needs. The archive is embedded in the image, producing a self-contained image. By default, the image starts MATLAB in an interactive command-line prompt. The image also installs the [matlab-proxy](https://github.com/mathworks/matlab-proxy) Python&reg; package using [pipx](https://pipx.pypa.io/), so you can override the entrypoint to access the MATLAB desktop from a web browser.

Use this solution to run MATLAB on a machine with an ARM64 (`aarch64`) processor. The image runs as a Linux container, so the host operating system can be any platform whose Docker engine runs Linux ARM64 containers, including Linux, Windows on ARM, and Apple silicon Macs. On x86-64 hardware, use the [`Dockerfile`](../../../Dockerfile) in the top-level repository instead.

## Requirements

* A machine with an ARM64 (`aarch64`) processor and a Docker engine that runs Linux ARM64 containers. The host operating system can be Linux, Windows on ARM, or an Apple silicon Mac; on Windows and macOS, use Docker Desktop, which runs Linux containers in a lightweight virtual machine.
* A Docker environment with [BuildKit enabled](https://docs.docker.com/build/buildkit/#getting-started) (default in current Docker releases). The build uses a bind mount so that the large MATLAB archive never becomes a layer in the final image.
* A working internet connection at build time. The build pulls the prebuilt matlab-deps base image from the GitHub Container Registry and installs matlab-proxy from the Python Package Index.
* The MATLAB ARM64 archive for this offering is already licensed, so you do not need to configure a license to run MATLAB.

## Get the Files

Clone this repository and navigate to the `matlab-linux-arm64` subfolder, which is the Docker build context.

```bash
git clone https://github.com/mathworks-ref-arch/matlab-dockerfile.git
cd matlab-dockerfile/alternates/matlab-linux-arm64
```

## Download the MATLAB ARM64 Archive

Download the Linux ARM64 build of MATLAB and place the `tar.gz` archive in the [`matlab-arm-image`](../matlab-arm-image) folder next to the Dockerfile. For instructions, see [`matlab-arm-image/README.md`](../matlab-arm-image/README.md). Do not extract the archive.

After this step, the folder contains the archive, for example:

```
matlab-arm-image/
├── README.md
└── MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz
```

If your archive has a different filename, pass it to the build with the `MATLAB_TGZ` build argument.

## Build Instructions

The `matlab-linux-arm64` folder is the Docker build context, so run `docker build` from there (not from this `build` folder).

The examples tag images as `matlab:<release>-<base>-<arch>`, for example `matlab:r2026b-ubuntu26.04-arm64`. Encoding the base image and architecture in the tag keeps variants that are built from the same release distinguishable. Use the OCI architecture name `arm64` (not `aarch64`). Because the MATLAB archive is currently arm64 only, these images are tagged per architecture rather than published as a multi-architecture manifest list.

### Build on the default base image

By default, the Dockerfile builds on the prebuilt matlab-deps image `ghcr.io/mathworks-ref-arch/container-images/matlab-deps-arm64:r2026b-ubuntu26.04`.

```bash
docker build -t matlab:r2026b-ubuntu26.04-arm64 .
```

### Choose a different base image

To build on a different operating system, set `BASE_IMAGE` to the matlab-deps-arm64 tag that most closely matches your needs. The available tags are published in the [matlab-deps-arm64 package](https://github.com/mathworks-ref-arch/container-images/pkgs/container/container-images%2Fmatlab-deps-arm64). For MATLAB R2026b, the tags include:

| `BASE_IMAGE` tag | Operating system |
|---|---|
| `r2026b-ubuntu26.04` (default) | Ubuntu 26.04 |
| `r2026b-ubuntu24.04` | Ubuntu 24.04 |
| `r2026b-ubuntu22.04` | Ubuntu 22.04 |
| `r2026b-debian13` | Debian 13 |
| `r2026b-debian12` | Debian 12 |
| `r2026b-ubi9` | Red Hat&reg; UBI 9 |
| `r2026b-ubi8` | Red Hat UBI 8 |

For example, to build on Red Hat UBI 9:

```bash
docker build \
  --build-arg BASE_IMAGE=ghcr.io/mathworks-ref-arch/container-images/matlab-deps-arm64:r2026b-ubi9 \
  -t matlab:r2026b-ubi9-arm64 .
```

The prebuilt image already provides the MATLAB operating-system dependencies, so the build only adds the extra packages that matlab-proxy needs (Python and pipx), plus the `Xvfb` headless X server and the `openbox` window manager that the matlab-proxy browser desktop requires.

The matlab-proxy browser desktop needs `Xvfb` and `openbox`. On apt-based (Ubuntu/Debian) images these are always installed. On RHEL-family images they come from AppStream/CRB (`Xvfb`) and EPEL (`openbox`), which are available on Rocky Linux, AlmaLinux, CentOS Stream, and subscribed RHEL. If they cannot be installed, the build still succeeds and prints a warning: command-line MATLAB and `-batch` work, but the browser desktop does not. See [Enable the matlab-proxy Desktop on Red Hat UBI](#enable-the-matlab-proxy-desktop-on-red-hat-ubi) for the UBI case.

### Enable the matlab-proxy Desktop on Red Hat UBI

Red Hat UBI ships only a subset of the RHEL repositories, which does not include `Xvfb` or the `epel-release` package, so a UBI image built with the defaults is command-line only. To build a UBI image that supports the browser desktop, set the `ADD_PROXY_DESKTOP_REPOS` build argument to `true`:

```bash
docker build \
  --build-arg BASE_IMAGE=ghcr.io/mathworks-ref-arch/container-images/matlab-deps-arm64:r2026b-ubi9 \
  --build-arg ADD_PROXY_DESKTOP_REPOS=true \
  -t matlab:r2026b-ubi9-arm64 .
```

With this argument, the build adds [EPEL](https://docs.fedoraproject.org/en-US/epel/) (for `openbox`) from its RPM URL and the [CentOS Stream](https://www.centos.org/centos-stream/) AppStream repository (for `Xvfb`), then installs both packages.

> **Note**
> This pulls the `Xvfb` package and its dependencies from CentOS Stream into a UBI-based image. Red Hat does not support mixing CentOS Stream packages into UBI, so use this option only if that trade-off is acceptable for your environment. On Ubuntu and Debian base images the browser desktop works without any extra arguments.

## Run Instructions

Images built with these instructions embed MATLAB, so you can run them directly without mounting a MATLAB installation.

The image uses `matlab` as its entrypoint, so arguments you pass to `docker run` are passed to MATLAB. Use the `--init` flag so that the container stops gracefully when a `docker stop` or `docker kill` command is issued. The MATLAB ARM64 archive for this offering is already licensed, so you do not need to configure a license to run the container.

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

The [Dockerfile](../Dockerfile) supports the following Docker build-time variables.

| Argument Name | Default value | Effect |
|---|---|---|
| `BASE_IMAGE` | `ghcr.io/mathworks-ref-arch/container-images/matlab-deps-arm64:r2026b-ubuntu26.04` | The prebuilt matlab-deps base image for the container. Set this to the [matlab-deps-arm64](https://github.com/mathworks-ref-arch/container-images/pkgs/container/container-images%2Fmatlab-deps-arm64) tag that matches your needs. See [Choose a different base image](#choose-a-different-base-image). |
| `MATLAB_RELEASE` | `R2026b` | The MATLAB release that the archive contains, in uppercase. For example: `R2026b`. |
| `MATLAB_TGZ` | `MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz` | The filename of the MATLAB ARM64 archive in the `matlab-arm-image` folder. |
| `PIP_INDEX_URL` | *unset* | A custom pip package index URL, for example a private PyPI mirror. Leave unset to install matlab-proxy from the public Python Package Index. |
| `ADD_PROXY_DESKTOP_REPOS` | *unset* | RHEL-family images only. Set to `true` to add EPEL and CentOS Stream AppStream so the matlab-proxy desktop dependencies (`Xvfb` and `openbox`) install on bases that lack them, notably Red Hat UBI. See [Enable the matlab-proxy Desktop on Red Hat UBI](#enable-the-matlab-proxy-desktop-on-red-hat-ubi). No effect on apt-based images. |

Use these arguments with the `docker build` command to customize your image. Alternatively, you can change the default values for these arguments directly in the [Dockerfile](../Dockerfile).

## More MATLAB Docker Resources
For more MATLAB Docker resources, see [More MATLAB Docker Resources](../../../README.md#more-matlab-docker-resources).

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
