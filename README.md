# RyzenAdj

Reworked fork of [RyzenAdj](https://github.com/FlyGoat/RyzenAdj), library only.

If running linux and using ryzen_smu, you will need this driver for this RyzenAdj fork: [CkNoSFeRaTU ryzen_smu driver](https://github.com/CkNoSFeRaTU/ryzen_smu).

## Build

### Build Requirements

Building this tool requires C & C++ compilers as well as cmake.

### Linux

RyzenAdj needs elevated access to the NB config space. This can be achieved by using either one of
these two methods:

* Using libpci and exposing `/dev/mem`
* Using the ryzen\_smu kernel module

RyzenAdj will try ryzen\_smu first, and then fallback to /dev/mem, if no compatible smu driver is found.  
The minimum supported version of ryzen_smu is 0.1.7  
If no backend is available, RyzenAdj will fail initialization.  

_**Please note that `/dev/mem` access may be restricted, for security reasons, in your kernel config**_

Please make sure that you have libpci dependency before compiling.

On Debian-based distros this is covered by installing **pcilib-dev** package:

    sudo apt install build-essential cmake libpci-dev

On Fedora:

    sudo dnf install cmake gcc-c++ pciutils-devel

On Arch:

    sudo pacman -S base-devel pciutils cmake


On OpenSUSE Tumbleweed:

    sudo zypper in cmake gcc14-c++ pciutils-devel

You may need to add the `iomem=relaxed` param to your kernel params on Tumbleweed, or [you may run into errors at runtime](https://github.com/FlyGoat/RyzenAdj/issues/241). 

If your Distribution is not supported, try finding the packages or use [Distrobox](https://github.com/89luca89/distrobox) or [Toolbox](https://docs.fedoraproject.org/en-US/fedora-silverblue/toolbox/) instead.

The simplest way to build it:

```sh
git clone https://github.com/PowerTuner/RyzenAdj.git
cd RyzenAdj
cmake -B build -DCMAKE_BUILD_TYPE=Release
make -C build
```

#### Ryzen\_smu

To let RyzenAdj use ryzen\_smu module, you have to install it first, it is not part of the linux kernel.

On Fedora:

```sh
sudo dnf install cmake gcc gcc-c++ dkms openssl
```

Clone and install ryzen\_smu:

```sh
git clone -b dev https://github.com/CkNoSFeRaTU/ryzen_smu
cd ryzen_smu/ && sudo make dkms-install
```

### Windows

Install latest [visual studio IDE community edition](https://visualstudio.microsoft.com/it/) and select **Desktop development in C++** in the installer.

Install [Windows terminal](https://apps.microsoft.com/detail/9n0dx20hk701) (the default terminal in Windows 11).

Open terminal and a new instance of **Developer Command Promp for VS xx** or **Developer PowerShell for VS xx**.

(where **xx** is the visual studio version)

```sh
git clone https://github.com/PowerTuner/RyzenAdj.git
cd RyzenAdj
cmake -B build -DCMAKE_BUILD_TYPE=Release -G "Ninja"
ninja -C build
```
