# MATLAB Linux ARM64 Archive

Place the Linux® ARM64 build of MATLAB® in this folder before building the Dockerfile in
the parent directory.

## Download the archive

> **Note**
> This is a placeholder. Replace the steps below with the official download link and
> instructions once they are available on the MathWorks website.

1. Go to the MathWorks website and download the Linux ARM64 MATLAB archive
   (for example `MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz`).
2. Copy the downloaded `.tar.gz` file into this `matlab-arm-image` folder, next to this
   README. Do not extract it.

The folder should then contain the archive:

```
matlab-arm-image/
├── README.md
└── MATLABR2026b_OpenBeta_aarch64_v3.0.tar.gz
```

## Notes

- The archive is a complete MATLAB root. The Dockerfile extracts it directly into the
  MATLAB install location (`/opt/matlab/<release>` by default).
- If the archive filename differs from the default, pass it to the build with
  `--build-arg MATLAB_TGZ=<filename>`.
- The `.gitignore` in this folder keeps the archive out of version control. Only this
  README is tracked, so the folder ships empty.
