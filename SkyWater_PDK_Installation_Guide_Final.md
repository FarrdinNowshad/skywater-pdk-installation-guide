# 🎓 Complete SkyWater 130nm PDK Installation Guide
## For Absolute Beginners - Ubuntu 22.04 LTS

**Author:** Farrdin Nowshad  
**Last Updated:** February 2026  
**Tested On:** Ubuntu 22.04 LTS (VirtualBox and Native)  
**Installation Time:** 2-4 hours  
**Disk Space Required:** ~100GB

---

## 📚 Table of Contents

1. [What You'll Install](#what-youll-install)
2. [Before You Start](#before-you-start)
3. [Pre-Installation Setup](#pre-installation-setup)
4. [Install All Tools](#install-all-tools)
5. [Install SkyWater PDK](#install-skywater-pdk)
6. [Configure Your Environment](#configure-your-environment)
7. [Verify Everything Works](#verify-everything-works)
8. [Common Problems & Solutions](#common-problems--solutions)
9. [Next Steps](#next-steps)

---

## 🎯 What You'll Install

This guide installs a **complete VLSI design toolchain** for chip design:

| Tool | Purpose | Why You Need It |
|------|---------|-----------------|
| **Magic** | Layout editor | Draw chip layouts |
| **Xschem** | Schematic editor | Draw circuit schematics |
| **Ngspice** | Circuit simulator | Simulate analog circuits |
| **Netgen** | LVS tool | Verify layout matches schematic |
| **KLayout** | Layout viewer | View and edit chip layouts |
| **Icarus Verilog** | Digital simulator | Simulate digital circuits |
| **GTKWave** | Waveform viewer | View simulation results |
| **Yosys** | RTL synthesis | Convert Verilog to gates |
| **SkyWater PDK** | Process Design Kit | 130nm fabrication data |

**What is a PDK?** A Process Design Kit contains all the files needed to design chips that can be manufactured. Think of it as the "rules and parts library" for making real silicon chips.

---

## 🚀 Before You Start

### System Requirements

- **Operating System:** Fresh Ubuntu 22.04 LTS installation
- **RAM:** Minimum 4GB (8GB recommended)
- **Disk Space:** 100GB free space (PDK and tools are large)
- **Internet:** Stable connection (will download ~3GB)
- **Time:** 2-4 hours (mostly automated)

### Important Notes

⚠️ **Run commands one section at a time**  
⚠️ **Don't skip sections** - order matters!  
⚠️ **Read error messages** - they usually tell you what's wrong  
⚠️ **Copy-paste carefully** - one wrong character breaks everything

### What "Terminal" Means
---
# SkyWater 130nm PDK — Installation Guide (Ubuntu 22.04 LTS)

Author: Farrdin Nowshad
Last updated: February 2026
Tested on: Ubuntu 22.04 LTS (VirtualBox and native)
Estimated time: 2–4 hours
Disk space required: ~100 GB

Table of contents
- What this guide installs
- Prerequisites
- Pre-installation steps
- Install required packages and tools
- Install the SkyWater (sky130) PDK
- Environment configuration
- Verification and smoke tests
- Troubleshooting
- Next steps

What this guide installs

This document describes how to prepare a complete open-source VLSI toolchain and install the SkyWater 130 nm PDK (sky130) on Ubuntu 22.04. The primary tools covered are:

- Magic — layout editor
- Xschem — schematic editor
- Ngspice — analog circuit simulator
- Netgen — LVS (layout vs schematic)
- KLayout — layout viewer/editor
- Icarus Verilog — Verilog simulator
- GTKWave — waveform viewer
- Yosys — RTL synthesis
- Open_PDKs / sky130 — process design kit (PDK)

Prerequisites

- Ubuntu 22.04 LTS (fresh install recommended)
- 4 GB RAM minimum; 8 GB recommended
- ~100 GB free disk space
- Stable internet connection (downloads ~2–4 GB)
- Sudo / administrative access

Notes

- Run commands sequentially and review output for errors.
- Copy/paste commands carefully; minor typos can cause failures.

Pre-installation

1) Confirm sudo access

```bash
sudo whoami
```

Expected: prints "root". If the account has no sudo privileges, either use an account with sudo or have an administrator add your user to the `sudo` group.

2) Update the system

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y software-properties-common curl git wget build-essential
```

3) (Optional but recommended) Create an 8 GB swap file if your system has limited RAM

```bash
sudo swapoff -a || true
sudo rm -f /swapfile
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
grep -q '/swapfile' /etc/fstab || echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
free -h
```

Install dependencies

Install the packages required to build and run the toolchain. Copy the following block and run it in a terminal.

```bash
sudo apt install -y \
    build-essential flex bison gawk tcl-dev tk-dev \
    libx11-dev libxrender-dev libxpm-dev \
    libncurses-dev blt freeglut3-dev mesa-common-dev libgl1-mesa-dev libglu1-mesa-dev \
    libgtk-3-dev libcairo2-dev libxaw7-dev fontconfig libxft-dev libxmu-dev libxext-dev \
    libreadline-dev libtool autoconf automake gettext \
    python3 python3-pip python3-dev python3-venv \
    cmake gfortran gcc m4 tcsh ruby \
    libboost-all-dev swig git \
    libfftw3-dev libsuitesparse-dev libblas-dev liblapack-dev \
    libglib2.0-dev libexpat1-dev libpng-dev zlib1g-dev \
    qt5-default qttools5-dev qttools5-dev-tools \
    libqt5widgets5 libqt5gui5 libqt5core5a libqt5svg5-dev libqt5x11extras5-dev \
    libgit2-dev gperf clang pkg-config xterm graphviz libeigen3-dev \
    libxcb1-dev libbz2-dev liblzma-dev
```

Create working directories

```bash
mkdir -p $HOME/sky130repo $HOME/tools $HOME/openpdk $HOME/workingdir/{layout,schematic,verilog,simulations}
```

Build and install tools

The order below is recommended. Each tool includes a short verification step.

1. Magic (layout editor)

```bash
cd $HOME/sky130repo
git clone https://github.com/RTimothyEdwards/magic.git magic-git
cd magic-git
./configure --prefix=$HOME/tools/magic
make -j$(nproc)
make install
$HOME/tools/magic/bin/magic --version
```

2. Xschem (schematic editor)

```bash
cd $HOME/sky130repo
git clone https://github.com/StefanSchippers/xschem.git xschem-git
cd xschem-git
./configure --prefix=$HOME/tools/xschem
make -j$(nproc)
make install
$HOME/tools/xschem/bin/xschem --version
```

3. Ngspice (analog simulator)

```bash
cd $HOME/sky130repo
wget https://sourceforge.net/projects/ngspice/files/ng-spice-rework/45.2/ngspice-45.2.tar.gz
tar -xzf ngspice-45.2.tar.gz
cd ngspice-45.2
./configure --prefix=$HOME/tools/ngspice --with-x --enable-xspice --enable-cider --with-readline=yes --enable-openmp --disable-debug
make -j$(nproc)
make install
$HOME/tools/ngspice/bin/ngspice --version
```

4. Netgen (LVS)

```bash
cd $HOME/sky130repo
git clone git://opencircuitdesign.com/netgen netgen-git
cd netgen-git
./configure --prefix=$HOME/tools/netgen
make -j$(nproc)
make install
$HOME/tools/netgen/bin/netgen --version
```

5. KLayout (layout viewer)

Install Qt packages first (if not already installed):

```bash
sudo apt update
sudo apt install -y qt5-default qttools5-dev qttools5-dev-tools libqt5widgets5 libqt5gui5 libqt5core5a libqt5svg5-dev libqt5x11extras5-dev
```

Then build:

```bash
cd $HOME/sky130repo
git clone https://github.com/KLayout/klayout.git klayout-git
cd klayout-git
rm -rf build-release
mkdir build-release
./build.sh -j$(nproc) -prefix $HOME/tools/klayout
mkdir -p $HOME/tools/klayout/bin
cp -r bin-release/* $HOME/tools/klayout/bin/
$HOME/tools/klayout/bin/klayout -v
```

6. Icarus Verilog

```bash
cd $HOME/sky130repo
git clone https://github.com/steveicarus/iverilog.git iverilog-git
cd iverilog-git
sh autoconf.sh
./configure --prefix=$HOME/tools/iverilog
make -j$(nproc)
make install
$HOME/tools/iverilog/bin/iverilog -V
```

7. GTKWave

```bash
sudo apt install -y gtkwave
gtkwave --version
```

8. Yosys

```bash
cd $HOME/sky130repo
git clone https://github.com/YosysHQ/yosys.git yosys-git
cd yosys-git
git submodule update --init
make -j$(nproc)
make install PREFIX=$HOME/tools/yosys
$HOME/tools/yosys/bin/yosys -V
```

Install the SkyWater / sky130 PDK

Clone and build Open_PDKs with the sky130 target. Magic must be available in PATH prior to configuring the PDK.

```bash
cd $HOME/sky130repo
rm -rf open_pdks
git clone --recurse-submodules https://github.com/RTimothyEdwards/open_pdks.git
cd open_pdks
export PATH=$HOME/tools/magic/bin:$PATH
./configure --enable-sky130-pdk --prefix=$HOME/openpdk
make -j$(nproc)
make install
ls $HOME/openpdk/share/pdk/sky130A/
```

Environment configuration

Append the required environment variables and PATH entries to `~/.bashrc` (or your preferred shell config). Example:

```bash
cat >> ~/.bashrc <<'EOF'
# SkyWater PDK and tools
export PDK_ROOT=$HOME/openpdk/share/pdk
export SKY130A=$HOME/openpdk/share/pdk/sky130A
export PATH=$HOME/tools/magic/bin:$HOME/tools/xschem/bin:$HOME/tools/ngspice/bin:$HOME/tools/netgen/bin:$HOME/tools/klayout/bin:$HOME/tools/iverilog/bin:$HOME/tools/yosys/bin:$PATH
export LD_LIBRARY_PATH=$HOME/tools/magic/lib:$HOME/tools/ngspice/lib:$LD_LIBRARY_PATH
EOF
source ~/.bashrc
```

Quick verification

Run simple checks to confirm the toolchain is available on PATH and the PDK directories exist:

```bash
which magic || echo 'magic not found'
which xschem || echo 'xschem not found'
which ngspice || echo 'ngspice not found'
ls $HOME/openpdk/share/pdk/sky130A/ || echo 'sky130 PDK not installed'
```

Troubleshooting (common issues)

- Command not found: Ensure you sourced `~/.bashrc` or opened a new terminal.
- Magic reports "technology not found": verify `$SKY130A` is set and `.magicrc` points to the sky130 tech files.
- PDK build fails with "need magic": ensure Magic is installed and on PATH before configuring Open_PDKs.
- KLayout build errors: make sure Qt development packages are installed.

Next steps

- Start with a simple schematic in Xschem and run Ngspice simulation.
- Create a layout in Magic and run LVS with Netgen.
- Try a minimal Verilog example with Icarus Verilog and inspect waveforms in GTKWave.

If you would like, I can:

- produce a condensed README.md suitable for GitHub with installation badges and quick-start snippets, or
- open a PR-style diff with the updated file content.

---
