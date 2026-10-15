# Build a MATLAB on Linux ARM64 Container Image

> **Most beta users do not need this page.** The simplest way to try this offering is to pull a prebuilt image and mount your MATLAB installation into it &mdash; see the [main README](../README.md). Use the instructions here only if you need or want to **build your own image**, for example to embed MATLAB in a self-contained image or to publish images to your own registry.

The [`Dockerfile`](../Dockerfile) in this folder builds a Docker&reg; image containing the Linux&reg; ARM64 build of MATLAB&reg;. Instead of installing MATLAB with the MATLAB Package Manager (*mpm*) or the installer, it unpacks a MATLAB `tar.gz` archive onto a prebuilt [MATLAB dependencies (matlab-deps)](https://github.com/mathworks-ref-arch/container-images/pkgs/container/container-images%2Fmatlab-deps-arm64) image, so the MATLAB operating-system dependencies are already present and you just pick the prebuilt base image that most closely matches your needs. The archive is embedded in the image, producing a self-contained image. The image runs `matlab` by default, so you can use it interactively on the command line or in `-batch` mode. See [Run Instructions](#run-instructions).

Use this solution to run MATLAB on a machine with an ARM64 (`aarch64`) processor. The image runs as a Linux container, so the host operating system can be any platform whose Docker engine runs Linux ARM64 containers, including Linux, Windows on ARM, and Apple silicon Macs.

## Requirements

* A machine with an ARM64 (`aarch64`) processor and a Docker engine that runs Linux ARM64 containers. The host operating system can be Linux, Windows on ARM, or an Apple silicon Mac; on Windows and macOS, use Docker Desktop, which runs Linux containers in a lightweight virtual machine.
* A Docker environment with [BuildKit enabled](https://docs.docker.com/build/buildkit/#getting-started) (default in current Docker releases). The build uses a bind mount so that the large MATLAB archive never becomes a layer in the final image.
* A working internet connection at build time, to pull the prebuilt matlab-deps base image from the GitHub Container Registry and the additional packages the build installs on top of it.
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

By default, the Dockerfile builds on the prebuilt matlab-deps image `ghcr.io/mathworks-ref-arch/container-images/matlab-deps-arm64:R2026b-ubuntu26.04`.

```bash
docker build -t matlab:r2026b-ubuntu26.04-arm64 .
```

### Choose a different base image

The Dockerfile builds on a prebuilt matlab-deps-arm64 base image. To build on a different operating system, set the `OS` build argument to one of the operating systems published in the [matlab-deps-arm64 package](https://github.com/mathworks-ref-arch/container-images/pkgs/container/container-images%2Fmatlab-deps-arm64):

| `OS` | Operating system |
|---|---|
| `ubuntu26.04` (default) | Ubuntu 26.04 |
| `ubuntu24.04` | Ubuntu 24.04 |
| `ubuntu22.04` | Ubuntu 22.04 |
| `debian13` | Debian 13 |
| `debian12` | Debian 12 |
| `ubi10` | Red Hat&reg; UBI 10 |
| `ubi9` | Red Hat UBI 9 |
| `ubi8` | Red Hat UBI 8 |

For example, to build on Red Hat UBI 9:

```bash
docker build --build-arg OS=ubi9 -t matlab:r2026b-ubi9-arm64 .
```

The prebuilt image already provides the MATLAB operating-system dependencies, so the build adds only a few extra packages, and which ones depend on the base image:

* On Ubuntu and Debian base images, the build installs Python and pipx (for matlab-proxy), plus the `Xvfb` headless X server and the `fluxbox` and `openbox` window managers that the matlab-proxy browser desktop requires. These images support the browser desktop.
* On Red Hat UBI base images, the build installs only `sudo` for the non-root `matlab` user. These images are **command-line only**: matlab-proxy and its browser-desktop dependencies (`Xvfb` and a window manager) are not installed, because those packages are not available in the Red Hat UBI repositories. Command-line MATLAB and `-batch` work the same as on the other images.

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

On Ubuntu and Debian images, [matlab-proxy](https://github.com/mathworks/matlab-proxy) serves the MATLAB desktop over HTTP. To use it, override the entrypoint with `matlab-proxy-app` and publish the matlab-proxy port.

```bash
docker run --init -it --rm -p 8888:8888 \
  --entrypoint matlab-proxy-app matlab:r2026b-ubuntu26.04-arm64
```

Open the URL that matlab-proxy prints in the terminal (by default `http://localhost:8888`) in a browser to access the MATLAB desktop.

## Customize the Image

The [Dockerfile](../Dockerfile) supports the following Docker build-time variables.

| Argument Name | Default value | Effect |
|---|---|---|
| `OS` | `ubuntu26.04` | The operating system of the matlab-deps-arm64 base image. See [Choose a different base image](#choose-a-different-base-image). |
| `MATLAB_TGZ` | `MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz` | The filename of the MATLAB ARM64 archive in the `matlab-arm-image` folder. |
| `PIP_INDEX_URL` | *unset* | A custom pip package index URL, for example a private PyPI mirror. Leave unset to install matlab-proxy from the public Python Package Index. |

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
