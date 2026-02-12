# SkyWater 130nm PDK Installation Guide

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/FarrdinNowshad/skywater-pdk-installation-guide/blob/main/LICENSE)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20LTS-E95420?logo=ubuntu)]([https://ubuntu.com/](https://releases.ubuntu.com/jammy/))
[![Tested](https://img.shields.io/badge/Tested-February%202026-success)](/)

**The most comprehensive, beginner-friendly guide to installing the complete SkyWater 130nm PDK toolchain on Ubuntu 22.04**

</div>

---

## Why This Guide?

After struggling through countless incomplete tutorials and scattered documentation, I created this guide to **save you hours of frustration**. This is the guide I wish I had when I started.

---

## 📊 Quick Stats

| Metric | Value |
|--------|-------|
| **Installation Time** | 4-6 hours |
| **Disk Space Required** | ~100GB |
| **Supported OS** | Ubuntu 22.04 LTS |
| **Tools Installed** | 9 essential VLSI tools |
| **Success Rate** | Verified on VirtualBox & Native |

---

## What You'll Install

| Tool | Purpose | Version |
|------|---------|---------|
| **Magic VLSI** | Layout editor for physical design | Latest |
| **Xschem** | Schematic capture and circuit editor | Latest |
| **Ngspice** | SPICE circuit simulator | 45.2 |
| **Netgen** | LVS (Layout vs Schematic) verification | Latest |
| **KLayout** | Advanced layout viewer and editor | 0.30.x |
| **Icarus Verilog** | Verilog HDL simulator | Latest |
| **GTKWave** | Waveform viewer | 3.3.x |
| **Yosys** | RTL synthesis framework | Latest |
| **SkyWater PDK** | 130nm open-source Process Design Kit | Latest |

---

## 🚀 Quick Start

### Prerequisites

- Fresh **Ubuntu 22.04 LTS** installation
- **100GB** free disk space
- **8GB RAM** recommended (4GB minimum)
- Stable internet connection

### One-Command Quick Install (Advanced Users)

```bash
# Clone this repository
git clone https://github.com/yourusername/skywater-pdk-installation-guide.git
cd skywater-pdk-installation-guide

# Run automated installation script (coming soon)
# ./install.sh
```

### Step-by-Step Installation (Recommended for Beginners)

**[Start with the Complete Guide](#-installation-steps)**

---

## Table of Contents

1. [System Requirements](#-system-requirements)
2. [Pre-Installation Setup](#-pre-installation-setup)
3. [Installation Steps](#-installation-steps)
   - [Magic VLSI](#1-magic-vlsi---layout-editor)
   - [Xschem](#2-xschem---schematic-editor)
   - [Ngspice](#3-ngspice---circuit-simulator)
   - [Netgen](#4-netgen---lvs-tool)
   - [KLayout](#5-klayout---layout-viewer)
   - [Icarus Verilog](#6-icarus-verilog---digital-simulator)
   - [GTKWave](#7-gtkwave---waveform-viewer)
   - [Yosys](#8-yosys---rtl-synthesis)
   - [SkyWater PDK](#9-skywater-130nm-pdk)
4. [Environment Configuration](#-environment-configuration)
5. [Verification](#-verification)
6. [Troubleshooting](#-troubleshooting)
7. [Next Steps](#-next-steps)

---

## System Requirements

### Minimum Requirements

- **OS:** Ubuntu 22.04 LTS (Desktop or Server)
- **CPU:** 2 cores (4+ recommended)
- **RAM:** 4GB (8GB recommended for compilation)
- **Disk:** 100GB free space
- **Internet:** Broadband connection (will download ~3GB)

### Tested Environments

 **VirtualBox** - Works perfectly with 8GB RAM allocated  
 **Native Ubuntu** - Best performance  
 **WSL2** - Not officially tested but should work  
 **Ubuntu 20.04** - May work but not tested

---

## Pre-Installation Setup

### Step 0: Fix Sudo Access (If Needed)

**Test sudo access:**
```bash
sudo whoami
```

If you see `root`, you're good! Skip to Step 1.

**If it fails** with "user is not in the sudoers file":

```bash
# Switch to root
su -

# Add your user to sudo group (replace 'username')
usermod -aG sudo username

# Exit and log back in
exit
```

Then log out and log back in completely.

---

### Step 1: Update System

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y software-properties-common curl git wget build-essential
```

---

### Step 2: Configure Swap Space

**Why?** Prevents system freezes during compilation of large tools.

**Check existing swap:**
```bash
sudo swapon --show
free -h
```

**If you have ≥8GB swap, skip this step.**

**Create 8GB swap:**
```bash
# Turn off existing swap (if any)
sudo swapoff /swapfile 2>/dev/null || true

# Remove old swap file
sudo rm -f /swapfile

# Create new 8GB swap
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
grep -q '/swapfile' /etc/fstab || echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verify
free -h
```

---

### Step 3: Install All Dependencies

**This is critical!** Install all dependencies at once to avoid issues later.

```bash
sudo apt install -y \
build-essential flex bison gawk tcl-dev tk-dev \
libx11-dev libx11-xcb-dev libxrender-dev libxpm-dev \
libncurses-dev libncurses5-dev blt freeglut3-dev \
mesa-common-dev libgl1-mesa-dev libglu1-mesa-dev \
libgtk-3-dev libgtk2.0-dev libcairo2 libcairo2-dev \
libxaw7 libxaw7-dev fontconfig libxft-dev libxmu6 libxmu-dev libxext-dev \
libreadline-dev libtool autoconf automake gettext \
python3 python3-pip python3-dev python3-venv \
cmake gfortran g++ gcc m4 tcsh csh ruby ruby-dev \
libboost-all-dev libboost-system-dev libboost-filesystem-dev \
libboost-thread-dev libboost-program-options-dev \
swig git-core \
libfftw3-dev libsuitesparse-dev libblas-dev liblapack-dev \
libglib2.0-dev libexpat1-dev libpng-dev zlib1g-dev \
qt5-default qtbase5-dev qttools5-dev qttools5-dev-tools \
libqt5widgets5 libqt5gui5 libqt5core5a \
libqt5multimedia5-plugins libqt5multimediawidgets5 qtmultimedia5-dev \
libqt5svg5-dev libqt5xmlpatterns5-dev libqt5designer5 libqt5x11extras5-dev \
libgit2-dev gperf clang libc++-dev libc++abi-dev \
libfl-dev pkg-config xterm xdot graphviz libeigen3-dev \
libxcb1-dev libxcb-render0-dev libxcb-shape0-dev libxcb-xfixes0-dev \
libbz2-dev liblzma-dev
```


---

### Step 4: Create Directory Structure

```bash
mkdir -p ~/sky130repo
mkdir -p ~/tools
mkdir -p ~/openpdk
mkdir -p ~/workingdir/{layout,schematic,verilog,simulations}
```

**What this creates:**
- `~/sky130repo/` - Tool source code
- `~/tools/` - Compiled tools
- `~/openpdk/` - SkyWater PDK
- `~/workingdir/` - Your design projects

---

## Installation Steps

> ** Total Time:** 4-6 hours (mostly automated compilation)

---

### 1. Magic VLSI - Layout Editor

**What it does:** Draw the physical layout of integrated circuits

```bash
cd ~/sky130repo

# Clone Magic
git clone https://github.com/RTimothyEdwards/magic magic-git
cd magic-git

# Fix readline issue (common bug)
if [ -d "scripts/readline-4.3" ]; then
    mv scripts/readline-4.3 scripts/readline
fi

# Build and install
./configure --prefix=$HOME/tools/magic
make -j$(nproc)
make install

# Verify
$HOME/tools/magic/bin/magic --version
```

**Expected output:** Version information  

---

### 2. Xschem - Schematic Editor

**What it does:** Create circuit schematics and netlists

```bash
cd ~/sky130repo

# Clone Xschem
git clone https://github.com/StefanSchippers/xschem.git xschem-git
cd xschem-git

# Build and install
./configure --prefix=$HOME/tools/xschem
make -j$(nproc)
make install

# Verify
$HOME/tools/xschem/bin/xschem --version
```

**Expected output:** Version information  

---

### 3. Ngspice - Circuit Simulator

**What it does:** Simulate analog and mixed-signal circuits

```bash
cd ~/sky130repo

# Download Ngspice 45.2
wget https://sourceforge.net/projects/ngspice/files/ng-spice-rework/45.2/ngspice-45.2.tar.gz
tar -xzf ngspice-45.2.tar.gz
cd ngspice-45.2

# Configure with all features
./configure --prefix=$HOME/tools/ngspice \
    --with-x \
    --enable-xspice \
    --enable-cider \
    --with-readline=yes \
    --enable-openmp \
    --disable-debug

# Build and install
make -j$(nproc)
make install

# Verify
$HOME/tools/ngspice/bin/ngspice --version
```

**Expected output:** `ngspice-45.2` 

---

### 4. Netgen - LVS Tool

**What it does:** Verify layout matches schematic (Layout vs Schematic)

```bash
cd ~/sky130repo

# Clone Netgen
git clone git://opencircuitdesign.com/netgen netgen-git
cd netgen-git

# Build and install
./configure --prefix=$HOME/tools/netgen
make -j$(nproc)
make install

# Verify
$HOME/tools/netgen/bin/netgen --version
```

**Expected output:** Version information 

---

### 5. KLayout - Layout Viewer

**What it does:** View, edit, and verify chip layouts

** Important:** Install Qt5 GUI packages first!

```bash
# Install Qt5 packages for GUI
sudo apt update
sudo apt install -y \
    qt5-default qttools5-dev qttools5-dev-tools \
    libqt5widgets5 libqt5gui5 libqt5core5a \
    libqt5multimedia5 libqt5multimediawidgets5 qtmultimedia5-dev \
    libqt5svg5-dev libqt5x11extras5-dev
```

**Now build KLayout:**

```bash
cd ~/sky130repo

# Remove old builds
rm -rf klayout-git

# Clone KLayout
git clone https://github.com/KLayout/klayout.git klayout-git
cd klayout-git

# Clean build directory
rm -rf build-release
mkdir build-release

# Build (this takes 20-30 minutes)
./build.sh -j$(nproc) -prefix $HOME/tools/klayout

# Copy executables
mkdir -p $HOME/tools/klayout/bin
cp -r bin-release/* $HOME/tools/klayout/bin/

# Verify
$HOME/tools/klayout/bin/klayout -v
```

**Expected output:** `KLayout 0.30.x`   

---

### 6. Icarus Verilog - Digital Simulator

**What it does:** Simulate digital circuits written in Verilog

```bash
cd ~/sky130repo

# Clone Icarus Verilog
git clone https://github.com/steveicarus/iverilog.git iverilog-git
cd iverilog-git

# Build and install
sh autoconf.sh
./configure --prefix=$HOME/tools/iverilog
make -j$(nproc)
make install

# Verify
$HOME/tools/iverilog/bin/iverilog -V
```

**Expected output:** `Icarus Verilog version 13.x`   

---

### 7. GTKWave - Waveform Viewer

**What it does:** Visualize simulation waveforms

**Recommended:** Install from package manager (easier and stable)

```bash
sudo apt install -y gtkwave
```

**Verify:**
```bash
gtkwave --version
```

**Expected output:** `GTKWave Analyzer v3.3.x`  

**Note:** You may see a harmless warning about "canberra-gtk-module" - ignore it.

---

### 8. Yosys - RTL Synthesis

**What it does:** Convert Verilog RTL to gate-level netlists

```bash
cd ~/sky130repo

# Clone Yosys
git clone https://github.com/YosysHQ/yosys.git yosys-git
cd yosys-git

# Configure
make config-gcc
echo "PREFIX := $HOME/tools/yosys" > Makefile.conf

# Update submodules
git submodule update --init

# Build (takes 10-15 minutes)
make -j$(nproc)
make install

# Verify
$HOME/tools/yosys/bin/yosys -V
```

**Expected output:** `Yosys 0.x`  

---

### 9. SkyWater 130nm PDK

**What it does:** Provides process technology files for 130nm chip fabrication

** CRITICAL:** Magic must be installed first (you already did this in Step 1)

**This downloads ~2GB and installs ~80GB of data. Takes 30-60 minutes.**

```bash
cd ~/sky130repo

# Remove any previous attempts
rm -rf open_pdks

# Clone with submodules (CRITICAL!)
git clone --recurse-submodules https://github.com/RTimothyEdwards/open_pdks.git

cd open_pdks

# Verify clone worked
ls -al
# You should see: README.md, configure, sky130/, etc.

# Initialize submodules (safety check)
git submodule update --init --recursive

# Ensure Magic is accessible
export PATH=$HOME/tools/magic/bin:$PATH
which magic
# Should show: /home/username/tools/magic/bin/magic

# Configure for Sky130
./configure \
    --enable-sky130-pdk \
    --prefix=$HOME/openpdk

# Build and install (downloads ~2GB, takes 30-60 minutes)
make -j$(nproc)
make install

cd ..
```

**Verify PDK installation:**
```bash
ls $HOME/openpdk/share/pdk/sky130A/
```

**Expected output:** `libs.ref` and `libs.tech` directories   

---

## Environment Configuration

**This step is CRITICAL!** Without it, tools won't find each other.

### Add Environment Variables

```bash
cat >> ~/.bashrc << 'EOF'

# ========== SkyWater PDK Tools ==========
export PDK_ROOT=$HOME/openpdk/share/pdk
export SKY130A=$HOME/openpdk/share/pdk/sky130A

# Tool binaries
export PATH=$HOME/tools/magic/bin:$PATH
export PATH=$HOME/tools/xschem/bin:$PATH
export PATH=$HOME/tools/ngspice/bin:$PATH
export PATH=$HOME/tools/netgen/bin:$PATH
export PATH=$HOME/tools/klayout/bin:$PATH
export PATH=$HOME/tools/iverilog/bin:$PATH
export PATH=$HOME/tools/yosys/bin:$PATH

# Library paths
export LD_LIBRARY_PATH=$HOME/tools/magic/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$HOME/tools/ngspice/lib:$LD_LIBRARY_PATH
# ========== End SkyWater PDK Tools ==========
EOF

# Apply changes
source ~/.bashrc
```

### Configure Magic

```bash
cp $SKY130A/libs.tech/magic/sky130A.magicrc ~/workingdir/layout/.magicrc
```

### Configure Xschem

```bash
cp $SKY130A/libs.tech/xschem/xschemrc ~/workingdir/schematic/xschemrc

cat >> ~/workingdir/schematic/xschemrc << 'EOF'
set XSCHEM_LIBRARY_PATH $env(SKY130A)/libs.tech/xschem
append XSCHEM_LIBRARY_PATH :$env(HOME)/tools/xschem/share/xschem/xschem_library
EOF
```

---

## Verification

Test that everything works:

### 1. Test Magic

```bash
cd ~/workingdir/layout
magic
```

In Magic console, type:
```
tech
```

**Expected:** Should display `sky130A`

Type `quit` to exit.

---

### 2. Test Xschem

```bash
cd ~/workingdir/schematic
xschem
```

Press `Shift + I` → Navigate to `sky130_fd_pr`  
**Expected:** Should see Sky130 transistors and components

---

### 3. Quick Test All Tools

```bash
magic --version
xschem --version
ngspice --version
netgen --version
klayout -v
iverilog -V
gtkwave --version
yosys -V

echo $PDK_ROOT
echo $SKY130A
```

**All commands should show version numbers without errors**

---

## 🐛 Troubleshooting

### Common Issues & Solutions

<details>
<summary><b>1. "Command not found" after installation</b></summary>

**Solution:**
```bash
source ~/.bashrc
# Or close terminal and open new one
```
</details>

<details>
<summary><b>2. Magic shows "technology not found"</b></summary>

**Solution:**
```bash
# Make sure you're in layout directory
cd ~/workingdir/layout

# Check .magicrc exists
ls -la .magicrc

# If missing:
cp $SKY130A/libs.tech/magic/sky130A.magicrc ~/workingdir/layout/.magicrc
```
</details>

<details>
<summary><b>3. Xschem doesn't show Sky130 devices</b></summary>

**Solution:**
```bash
# Check environment variable
echo $SKY130A

# If empty:
source ~/.bashrc

# Verify xschemrc
cat ~/workingdir/schematic/xschemrc | grep SKY130A
```
</details>

<details>
<summary><b>4. KLayout won't build - Qt errors</b></summary>

**Solution:**
```bash
sudo apt install -y qt5-default qttools5-dev libqt5widgets5 libqt5multimedia5 qtmultimedia5-dev

cd ~/sky130repo/klayout-git
rm -rf build-release
./build.sh -j$(nproc) -prefix $HOME/tools/klayout
```
</details>

<details>
<summary><b>5. PDK installation fails: "need magic"</b></summary>

**Solution:**
```bash
# Make sure Magic is in PATH
export PATH=$HOME/tools/magic/bin:$PATH
magic --version

# Retry PDK installation
cd ~/sky130repo/open_pdks
./configure --enable-sky130-pdk --prefix=$HOME/openpdk
make -j$(nproc)
make install
```
</details>

<details>
<summary><b>6. Ngspice download 404 error</b></summary>

**Solution:**
The version link may have changed. Check latest version at:
https://sourceforge.net/projects/ngspice/files/ng-spice-rework/

Update the version number in the wget command.
</details>

**More issues?** [Open an issue](https://github.com/yourusername/skywater-pdk-installation-guide/issues) with:
- Error message (full text)
- Command you ran
- Ubuntu version: `lsb_release -a`
- Output of: `uname -a`

---

## Next Steps - Start Designing!

### 1. Learn Magic VLSI

```bash
cd ~/workingdir/layout
magic
```

**Resources:**
- [Magic Tutorial](http://opencircuitdesign.com/magic/tutorials/)
- [Magic Command Reference](http://opencircuitdesign.com/magic/commandref/)

---

### 2. Create Schematics in Xschem

```bash
cd ~/workingdir/schematic
xschem
```

**Resources:**
- [Xschem Tutorial PDF](https://xschem.sourceforge.io/stefan/xschem_man/tutorial_xschem_slides.pdf)
- [Xschem Documentation](https://xschem.sourceforge.io/stefan/index.html)

---

### 3. Run SPICE Simulations

Create a simple testbench in Xschem:
- Tools → Netlist
- Tools → Simulate (launches Ngspice)
- View waveforms in Xschem or GTKWave

**Resources:**
- [Ngspice Manual](http://ngspice.sourceforge.net/docs.html)
- [SPICE Tutorial](http://bwrcs.eecs.berkeley.edu/Classes/IcBook/SPICE/)

---

### 4. Digital Design with Verilog

```bash
cd ~/workingdir/verilog
```

Create `test.v`:
```verilog
module test;
  initial begin
    $display("Hello, SkyWater PDK!");
    $finish;
  end
endmodule
```

Simulate:
```bash
iverilog -o test test.v
vvp test
```

**Resources:**
- [Verilog HDL Tutorial](https://hdlbits.01xz.net/)
- [Icarus Verilog Documentation](http://iverilog.icarus.com/)

---

### 5. Explore Example Designs

- [SkyWater PDK Examples](https://github.com/google/skywater-pdk)
- [OpenLane Flow](https://github.com/The-OpenROAD-Project/OpenLane)
- [Caravel User Project](https://github.com/efabless/caravel_user_project)

---

## Additional Resources

### Official Documentation

- [SkyWater PDK Docs](https://skywater-pdk.readthedocs.io/)
- [Magic VLSI](http://opencircuitdesign.com/magic/)
- [Xschem](https://xschem.sourceforge.io/)
- [Ngspice](http://ngspice.sourceforge.net/)
- [KLayout](https://www.klayout.de/)
- [Yosys](https://yosyshq.net/yosys/)

---

## 📄 License

This guide is licensed under the [MIT License](LICENSE).

The tools installed have their own licenses:
- Magic VLSI: MIT-like license
- Xschem: GPL v2
- Ngspice: BSD/GPL mixed
- SkyWater PDK: Apache 2.0

Please check individual tool licenses before use.

---

<div align="center">

**Made with for the open-source silicon community**

[⬆ Back to Top](#-skywater-130nm-pdk-installation-guide)

</div>
