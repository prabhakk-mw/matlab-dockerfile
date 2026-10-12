# Run MATLAB on Linux ARM64 in a Container

Run the Linux&reg; ARM64 build of MATLAB&reg; in a Docker&reg; container using a **prebuilt image**. You download MATLAB from the Open Beta, extract it, and mount it into a ready-to-run container &mdash; no image building required. The container starts MATLAB in a command-line prompt by default, and also includes [matlab-proxy](https://github.com/mathworks/matlab-proxy) so you can open the MATLAB desktop in a web browser.

This is the simplest way to try the Open Beta. If you would rather build your own image (for example, to embed MATLAB in a self-contained image or to use a different base operating system), see [Build Your Own Image](#build-your-own-image).

## Requirements

* A machine with an ARM64 (`aarch64`) processor and a Docker engine that runs Linux ARM64 containers. The host operating system can be Linux, Windows on ARM, or an Apple silicon Mac; on Windows and macOS, use Docker Desktop, which runs Linux containers in a lightweight virtual machine.
* The MATLAB ARM64 archive from the Open Beta. This archive is already licensed, so you do not need to configure a license.

## Step 1: Get the MATLAB Archive

Download the Linux ARM64 build of MATLAB (`tar.gz` archive) through the Open Beta process.

## Step 2: Extract MATLAB onto Your System

Extract the archive into `/opt/matlab/R2026b` on the host. Using this path means the host and container paths match, which keeps the mount in the next step simple.

```bash
sudo mkdir -p /opt/matlab/R2026b
sudo tar -xf MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz -C /opt/matlab/R2026b
```

## Step 3: Run the Container

Run the prebuilt image and mount your extracted MATLAB at the same path inside the container. Docker pulls the image automatically the first time.

```bash
docker run --init -it --rm \
  -v /opt/matlab/R2026b:/opt/matlab/R2026b \
  ghcr.io/mathworks-ref-arch/matlab-dockerfile/matlab-arm64:r2026b-ubuntu26.04
```

This starts MATLAB in an interactive command-line prompt. Use the `--init` flag so that the container stops gracefully when a `docker stop` or `docker kill` command is issued. You can mount MATLAB read-only by appending `:ro` to the volume.

That is all you need to get up and running. The sections below show other ways to run the container.

## Choose an Image for Your Operating System

Prebuilt images are published for several base operating systems, following the naming scheme `matlab-arm64:<release>-<os>`. Pick the tag that most closely matches your needs:

| Image tag | Operating system | Browser desktop |
|---|---|---|
| `r2026b-ubuntu26.04` | Ubuntu 26.04 | Yes |
| `r2026b-ubuntu24.04` | Ubuntu 24.04 | Yes |
| `r2026b-ubuntu22.04` | Ubuntu 22.04 | Yes |
| `r2026b-debian13` | Debian 13 | Yes |
| `r2026b-debian12` | Debian 12 | Yes |
| `r2026b-ubi9` | Red Hat&reg; UBI 9 | No |
| `r2026b-ubi8` | Red Hat UBI 8 | No |

Command-line MATLAB and `-batch` work on every image. The "Browser desktop" column indicates whether the prebuilt image also supports [running the MATLAB desktop in a browser with matlab-proxy](#run-the-matlab-desktop-in-a-browser-with-matlab-proxy). The prebuilt UBI images are command-line only, because the required `Xvfb` and `openbox` packages are not in the Red Hat UBI repositories. You can still enable the browser desktop on UBI by building your own image with those repositories added &mdash; see [Enable the matlab-proxy Desktop on Red Hat UBI](build/README.md#enable-the-matlab-proxy-desktop-on-red-hat-ubi).

Use the tag in place of `r2026b-ubuntu26.04` in the `docker run` commands, for example `ghcr.io/mathworks-ref-arch/matlab-dockerfile/matlab-arm64:r2026b-ubi9`.

## Run a MATLAB Command in Batch Mode

The image uses `matlab` as its entrypoint, so arguments you pass to `docker run` are passed to MATLAB. To start the container, run a MATLAB command, and then exit, pass `-batch` and the command. For example, to run `ver`, which displays the version of MATLAB and other installed products:

```bash
docker run --init --rm \
  -v /opt/matlab/R2026b:/opt/matlab/R2026b \
  ghcr.io/mathworks-ref-arch/matlab-dockerfile/matlab-arm64:r2026b-ubuntu26.04 -batch ver
```

## Run the MATLAB Desktop in a Browser with matlab-proxy

The image includes [matlab-proxy](https://github.com/mathworks/matlab-proxy), which serves the MATLAB desktop over HTTP. To use it, override the entrypoint with `matlab-proxy-app` and publish the matlab-proxy port.

> **Note**
> The prebuilt Red Hat UBI images (`r2026b-ubi9`, `r2026b-ubi8`) are command-line only and do not include the browser desktop, because the required `Xvfb` and `openbox` packages are not in the UBI repositories. Use an Ubuntu or Debian image for the browser desktop, or build your own UBI image with the desktop enabled &mdash; see [Enable the matlab-proxy Desktop on Red Hat UBI](build/README.md#enable-the-matlab-proxy-desktop-on-red-hat-ubi).

```bash
docker run --init -it --rm -p 8888:8888 \
  -v /opt/matlab/R2026b:/opt/matlab/R2026b \
  --entrypoint matlab-proxy-app \
  ghcr.io/mathworks-ref-arch/matlab-dockerfile/matlab-arm64:r2026b-ubuntu26.04
```

Open the URL that matlab-proxy prints in the terminal (by default `http://localhost:8888`) in a browser to access the MATLAB desktop.

## Build Your Own Image

If you need or want to build the image yourself instead of using a prebuilt image &mdash; for example, to embed MATLAB in a self-contained image, to build on a different base operating system, or to publish images to your own registry &mdash; see [`build/README.md`](build/README.md).

## More MATLAB Docker Resources
For more MATLAB Docker resources, see [More MATLAB Docker Resources](../../README.md#more-matlab-docker-resources).

## Help Make MATLAB Even Better
You can help improve MATLAB by providing user experience information on how you use MathWorks products. Your participation ensures that you are represented and helps us design better products. To opt out of this service, set the environment variable `MW_DDUX_FORCE_ENABLE` to `false` when you run the container (`-e MW_DDUX_FORCE_ENABLE=false`).

To learn more, see the documentation: [Help Make MATLAB Even Better - Frequently Asked Questions](https://www.mathworks.com/support/faq/user_experience_information_faq.html).

## Feedback
We encourage you to try this repository with your environment and provide feedback. If you encounter a technical issue or have an enhancement request, create an issue [here](https://github.com/mathworks-ref-arch/matlab-dockerfile/issues).

---
Copyright 2026 The MathWorks, Inc.
