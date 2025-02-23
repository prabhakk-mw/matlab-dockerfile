# MATLAB Package Manager

## Description

MATLAB® Package Manager (`mpm`) is a command-line package manager for installing MATLAB, Simulink®, and other MathWorks® products or support packages.

To get started:
1. Download `mpm` for your operating system: [Linux](#linux) | [Windows](#windows) | [macOS](#macos)
2. Run `mpm` from the command line or from a Dockerfile.

## Download MATLAB Package Manager
### Linux
Verify that this required software is installed:
* All third-party packages required to run the `mpm` command: `unzip`, `ca-certificates`
* All MATLAB dependencies. To view the list of dependencies, in the [MATLAB Dependencies](https://github.com/mathworks-ref-arch/container-images/tree/master/matlab-deps) repository, open the `<release>/<system>/base-dependencies.txt` file for your MATLAB release and operating system.

From a Linux terminal, use `wget` to download the latest version of `mpm`.

    wget https://www.mathworks.com/mpm/glnxa64/mpm

Give the downloaded file executable permissions so that you can run `mpm`.

    chmod +x mpm

### Windows
From a Windows PowerShell command prompt, use `wget` to download the latest version of `mpm`.

    wget https://www.mathworks.com/mpm/win64/mpm -outfile mpm.exe

> **Note**: You must run `mpm` from the Windows command prompt as an administrator or you get an error during installation.

### macOS
From a macOS terminal, use `curl` to download the latest version of `mpm` for your macOS architecture.

* macOS (Intel processor):

      curl -L -o https://www.mathworks.com/mpm/maci64/mpm

* macOS (Apple silicon):

      curl -L -o https://www.mathworks.com/mpm/maca64/mpm


Give the downloaded file executable permissions so that you can run `mpm`.

    chmod +x mpm

## Syntax

### Install Products

`mpm install --release=<release> --products <product1> ... <productN>` installs products `<product1> ... <productN>` from release `<release>` to the default installation folder. [Example](#install-products-to-default-folder)

`mpm install --release=<release> --products <product1> ... <productN> <installOptions>` sets additional [product installation options](#product-installation-options). For example, you can specify the install source or destination, whether to install documentation and examples, and whether to install the GPU libraries for use with Parallel Computing Toolbox™. [Example](#install-products-using-optional-command-line-inputs)

`mpm install --inputfile </full/path/to/file>` install products using the `</full/path/to/file>` input file. You can download a template input file for your desired release from the [mpm-input-files](mpm-input-files) folder. You must specify `--inputfile` without any other options. [Example](#install-products-using-input-file)

#### Product Installation Options

| Option | Description |
| ------ | ----------- |
`--release <release>` | <p>Release to install.</p><ul><li>To install the latest version of a release, specify the release name. Example: `R2023b`</li><li>To install a specific update release, specify the release name with an update number suffix. Example: `R2023bU4`</li><li>To install a release without updates, specify the release name with an update 0 or general release suffix. Example: `R2023bU0`, `R2023bGR`</li></ul><p>**Example**: `--release R2023b`</p>
`--products <product1 ... productN>` | <p>Products to install, specified as a list of product names separated by spaces.</p><p>`mpm` can install most MathWorks products and support packages. For the full list of correctly formatted product names, download the template input file for your desired release from the [mpm-input-files](mpm-input-files) folder and view the product and support package lists.</p><p>You do not need to specify all required products. If a product or support package requires another product to be installed, `mpm` installs it automatically.</p><p>For information on products `mpm` cannot install, see [Limitations](#limitations).</p><p>**Example:** `--products MATLAB Simulink Fixed-Point_Designer` installs MATLAB, Simulink, and Fixed-Point Designer.</p><p>**Example:** `--products Deep_Learning_Toolbox` installs Deep Learning Toolbox and also installs its required product, MATLAB, automatically.</p>
`--inputfile </full/path/to/file>` | <p>Full path to the input file used to install products.</p><p>Download a template input file for your desired release from the [mpm-input-files](mpm-input-files) folder and customize it for your installation. For example, you can specify the products and support packages to install and the desired installation folder.</p><p>You must specify `--inputfile` without any other options.</p><p>**Example:** `--inputfile /home/<USER>/matlab/mpm_input_r2023b.txt`</p>
`--destination </full/path/to/destination>` | <p>Full path to the installation destination folder.</p><p>If you are adding products or support packages to an existing MATLAB installation, specify the full path to where MATLAB is installed. `mpm` determines the folder to which to install support packages based on the MATLAB installation folder.</p><p>If you do not set `--destination`, then `mpm` installs to these locations by default, where `<release>` is the specified `--release` option.</p><p>**Linux:** `/usr/share/matlab`</p><p>**Windows:** `C:\Program Files\MATLAB\<release>`<ul><li>If the Windows machine already includes a MATLAB installation for the specified release, then `mpm` uses the installation folder of that MATLAB release as the default destination.</li></ul></p><p>**macOS:** `/Applications`</p> 
`--source </full/path/to/source>` | <p>Full path to the installation source. You can specify one of these sources:</p><ul><li>**Downloaded product files.** For more details, see [Download Products Without Installing](https://www.mathworks.com/help/install/ug/download-without-installing.html). *(R2018b and later releases)*</li><li>**An ISO or DMG image.** You can download images from [MathWorks Downloads](https://www.mathworks.com/downloads). *(R2021b and later releases)*</li></ul><p>If you do not set `--source`, then `mpm` downloads the the product files from MathWorks.</p>
`--doc` | <p>Flag to install documentation and examples. *(R2022b and earlier releases)*</p><p>To install the documentation in R2023a and later, see [Install Documentation](#install-documentation).</p>
`--no-gpu` | <p>Flag to skip installation of GPU libraries when you install Parallel Computing Toolbox. *(R2023a and later releases)*</p><p>If you do not intend to use GPU computing in MATLAB, specify this option to reduce the size of the install. You can install the GPU libraries later by calling a GPU function such as `gpuArray` or `gpuDevice` in MATLAB.</p>

### Install Documentation
*R2023a and later releases*

`mpm install-doc --matlabroot <matlabroot>` installs documentation and examples for the MATLAB installation at `<matlabroot>`. [Example](#install-products-using-optional-command-line-inputs)

`mpm install-doc --matlabroot <matlabroot> <docOptions>` sets additional [documentation installation options](#documentation-installation-options). For example, you can specify the path to a mounted ISO image of the documentation or the documentation installation destination. [Example](#install-products-using-optional-command-line-inputs)

#### Documentation Installation Options

| Option | Description |
| ------ | ----------- |
`--matlabroot </full/path/to/matlabroot>` | <p>Full path to the folder in which MATLAB is installed.</p>
`--source </full/path/to/source>` | <p>Full path to the mounted documentation ISO.</p><p>If you do not set `--source`, then `mpm` installs the documentation to `$PWD/archives`.</p><p>To download a documentation ISO, see [Install Documentation on Offline Machines](https://www.mathworks.com/help/install/ug/install-documentation-on-offline-machines.html).</p>
`--docroot </full/path/to/docroot>` | <p>Full path to the documentation installation folder.</p>

### Get Help and Version Information

`mpm --help` or `mpm -h` displays command-line help for `mpm`. Example: `./mpm --help`

`mpm --version` or `mpm -v` displays `mpm` version information. Example: `./mpm --version`

## Examples

### Install Products to Default Folder

Install MATLAB R2023b, Simulink, and Signal Processing Toolbox™ to the default folder. Navigate to the folder containing the `mpm` binary file and run this command.

* Linux or macOS:

      ./mpm install --release=R2023b --products MATLAB Simulink Signal_Processing_Toolbox
* Windows *(run as administrator)*:

      .\mpm.exe install --release=R2023b --products MATLAB Simulink Signal_Processing_Toolbox


You can install additional products later. For example, add Robotics System Toolbox™ to the MATLAB installation.

* Linux or macOS:

      ./mpm install --release=R2023b --products Robotics_System_Toolbox

* Windows *(run as administrator)*:

      .\mpm.exe install --release=R2023b --products Robotics_System_Toolbox
    
### Install Products Using Optional Command-Line Inputs

Install MATLAB R2023b and specify the installation destination folder. Also install Parallel Computing Toolbox but without the GPU libraries.

* Linux or macOS:
      
      ./mpm install --release=R2023b --destination=/home/<USER>/matlab --products MATLAB Parallel_Computing_Toolbox --no-gpu  

* Windows *(run as administrator)*:

      .\mpm.exe install --release=R2023b --destination=\users\<USER>\matlab --products MATLAB Parallel_Computing_Toolbox --no-gpu

Download a documentation ISO from [Install Documentation on Offline Machines](https://www.mathworks.com/help/install/ug/install-documentation-on-offline-machines.html) and mount the ISO. Install the documentation and examples, specifying the MATLAB installation folder and the path to the mounted ISO.

* Linux or macOS:

      ./mpm install-doc --matlabroot=/home/<USER>/matlab --source=/path/to/source

* Windows *(run as administrator)*:

      .\mpm.exe install-doc --matlabroot=\users\<USER>\matlab --source=\path\to\source

### Install Products Using Input File

Install products and support packages for MATLAB R2023b by specifying installation options in a file.

From the [mpm-input-files](mpm-input-files) folder, open the `R2023b` folder and download the `mpm_input_r2023b.txt` input file.

Open the file. Configure the MATLAB installation by uncommenting lines that start with a single `'#'` and updating their values. Update these sections:

*SPECIFY DESTINATION FOLDER*

Uncomment the `destinationFolder` line and set an installation folder. For example:

Linux or macOS:
```
destinationFolder=/home/<USER>/matlab
```
Windows:
```
destinationFolder=\users\<USER>\matlab
```

*INSTALL PRODUCTS*

Uncomment the `product.MATLAB` and `product.Simulink` lines for installing MATLAB And Simulink.

```
product.MATLAB
# ...
product.Simulink
```

*INSTALL SUPPORT PACKAGES*

Uncomment the `product.Deep_Learning_Toolbox_Model_for_ResNet-50_Network` line to install a Deep Learning Toolbox™ support package. `mpm` will automatically install this support package's required product, Deep Learning Toolbox.
```
product.Deep_Learning_Toolbox_Model_for_ResNet-50_Network
```
Save the file.

Install the products and support package.

* Linux or macOS:

      ./mpm install --inputfile /path/to/file/mpm_input_r2023b.txt

* Windows *(run as administrator)*:

      .\mpm.exe install --inputfile \path\to\file\mpm_input_r2023b.txt

## Limitations

- `mpm` supports installing products and support packages for these releases only:
  - Products - R2017b or later
  - Support Packages - R2019a or later
- Not all MathWorks products are available for all operating systems and architectures that MATLAB supports:
  - [Products Not Available for Linux](https://www.mathworks.com/support/requirements/matlab-linux.html)
  - [Products Not Available for Mac](https://www.mathworks.com/support/requirements/matlab-mac.html)
  - [Products Not Available for Apple silicon Macs](https://www.mathworks.com/support/requirements/apple-silicon.html)
- `mpm` cannot install these products:

  - IEC Certification Kit
  - DO Qualification Kit
  - Simulink Code Inspector™

  For alternative ways to install these products, see [Install Products](https://www.mathworks.com/help/install/install-products.html).

- `mpm` cannot install these support packages:

  - Image Acquisition Toolbox™ Support Package for GenICam™ Interface
  - Image Acquisition Toolbox Support Package for GigE Vision® Hardware
  - Simulink Coder™ Support Package for BBC micro:bit
  - MATLAB Support Package for IP Cameras
  - New Desktop for MATLAB
  - MATLAB Support Package for Parrot® Drones
  - MATLAB Support Package for Ryze Tello Drones
  - Simulink Real-Time™ Target Support Package

  To install these support packages within MATLAB, see [Get and Manage Add-Ons](https://www.mathworks.com/help/matlab/matlab_env/get-add-ons.html).

- On Windows, uninstalling products that were installed using `mpm` is not supported.

## Feedback and Support

If you encounter a technical issue or have an enhancement request, create an issue [here](https://github.com/mathworks-ref-arch/matlab-dockerfile/issues).

## Changelog

### 2023.12.1 - December 14, 2023
- **Added**: Support for Windows
- **Added**: Support for macOS

### 2023.10.0.1 - October 26, 2023
- **Added**: Install hardware and software support packages.
- **Added**: Install required products automatically. For example, if you specify `--product Simulink`, then `mpm` installs both Simulink and the required product MATLAB.
- **Added**: Install products by specifying options in an input file.

### 2023.9 - September 13, 2023

- **Fixed:** Resolved an issue that prevented installation of R2023b.

### 2023.3 - March 23, 2023

- **Added:** Specify the `--no-gpu` option to prevent installation of GPU libraries when you install Parallel Computing Toolbox.
- **Changed:** Versioning changed from [Semantic Versioning](https://semver.org/) to [Calendar Versioning](https://calver.org/). 

### 0.8.0 - December 17, 2022

- **Added:** Install a specific MATLAB update level. For example, to download MATLAB R2022b Update 2, specify the `--release` option as `R2022bU2`.

### 0.7.0 - November 4, 2022

- **Added:** Foundation changes to support upcoming features.
- **Fixed:** Improved parsing for `--release` option.

### 0.6.0 - July 14, 2022

- **Added:** By default, MATLAB documentation and examples are not installed with MATLAB.
- **Added:** Use the `--doc` option to include documentation and examples with the MATLAB installation.
- **Added:** The `mpm` command now observes `TMPDIR` environment variable.
- **Fixed:** The `mpm` command no longer crashes at run time on RHEL7/UBI7.
- **Fixed:** Changed dropped packet behavior to improve download resilience in adverse network conditions.
- **Fixed:** Improved error message for installing into a MATLAB instance that does not match the specified release.

### 0.5.1 - February 14, 2022

- **Fixed:** Installing a toolbox into an existing MATLAB instance at any update level that is not the latest update level for that release no longer results in the installed products throwing an error when invoked.

### 0.5.0 - December 6, 2021

- **Added:** Initial release
