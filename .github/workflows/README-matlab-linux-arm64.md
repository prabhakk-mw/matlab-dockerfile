# Build, Test, and Publish the Linux ARM64 Dockerfile

> This folder is intended only for the administrators of the matlab-dockerfile repository.

The workflow in this folder builds, tests, and publishes the prebuilt Linux ARM64 images for the Dockerfile found in [`alternates/matlab-linux-arm64`](../../alternates/matlab-linux-arm64).

These images do **not** embed MATLAB. Users pull an image and mount their own extracted MATLAB installation at run time, as described in the [alternate's README](../../alternates/matlab-linux-arm64/README.md).

The Red Hat UBI base images (`ubi9`, `ubi8`) are command-line only: the build does not install matlab-proxy or its browser-desktop dependencies (`Xvfb` and a window manager) on them, because those packages are not available in the UBI repositories. Command-line MATLAB and `-batch` work on these images. matlab-proxy and the browser desktop are available on the Ubuntu and Debian images only.

## Triggers and Scheduled Jobs

The workflow is scheduled to run every Monday at 00:00. The weekly rebuild picks up new `matlab-proxy` releases from the Python Package Index and updates to the `matlab-deps` base images.

Additionally, the workflow is triggered each time you push a change to the [`Dockerfile`](../../alternates/matlab-linux-arm64/Dockerfile) or to the workflow file itself.

You can also trigger the workflow from the "Actions" tab.

## Push the Image to the GitHub Container Registry

Each time an image passes the smoke test, it is pushed to the GitHub Container Registry with the name:

`ghcr.io/mathworks-ref-arch/matlab-dockerfile/matlab-arm64:${matlab-release}-${os}`

where `${matlab-release}` is a lowercase MATLAB release (for example `r2026b`) and `${os}` is the base operating system (for example `ubuntu26.04` or `ubi9`). The `${matlab-release}-${os}` combination matches the corresponding [`matlab-deps-arm64`](https://github.com/mathworks-ref-arch/container-images/pkgs/container/container-images%2Fmatlab-deps-arm64) base image tag.

## Workflow Description

The workflow runs a one-dimensional matrix of jobs, one per base operating system. To ensure that a failure in any job does not cancel the others, the `fail-fast` option is set to `false`. The jobs run on a native ARM64 runner (`ubuntu-24.04-arm`), which GitHub provides for public repositories.

Each job consists of the following steps:

1. Check out the repository into a GitHub Actions runner.
2. Free up disk space on the runner.
3. Compute the base image tag and the published image tag from the MATLAB release and operating system.
4. Log in to ghcr.io, the GitHub Container Registry.
5. Build the image locally, using the matching `matlab-deps-arm64` image as the base. The build context does not contain a MATLAB archive, so MATLAB is not embedded.
6. Smoke-test the image: confirm that the `matlab` launcher symlink is present and, on the Ubuntu and Debian images, that `matlab-proxy-app` is on the `PATH`. The UBI images are command-line only, so the `matlab-proxy-app` check is skipped for them.
7. Push the image to the GitHub Container Registry if it passed the smoke test.

---

Copyright 2026 The MathWorks, Inc. All rights reserved.
