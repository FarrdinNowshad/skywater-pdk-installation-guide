# Complete SkyWater 130nm PDK Installation Guide
## For Ubuntu 22.04 LTS

**Author:** Farrdin Nowshad  
**Last Updated:** February 2026  
**Tested On:** Ubuntu 22.04 LTS (VirtualBox and Native)  
**Installation Time:** 4-6 hours  
**Disk Space Required:** ~100GB


## What You'll Install

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

---

## Before You Start

### System Requirements

- **Operating System:** Fresh Ubuntu 22.04 LTS installation
- **RAM:** Minimum 4GB (8GB recommended)
- **Disk Space:** 100GB free space (PDK and tools are large)
- **Internet:** Stable connection (will download ~3GB)

---

## Pre-Installation Setup

### Step 0: Fix Sudo Access (If Needed)

**What is sudo?** It's like "Run as Administrator" in Windows. You need it to install software.

**Do you have sudo access?** Test it:
```bash
sudo whoami
```

**If it works:** You'll see `root` - skip to Step 1.

**If it fails** with "user is not in the sudoers file":

1. **Switch to root user:**
   ```bash
   su -
   ```
   Enter the root password when asked.

2. **Add yourself to sudo group** (replace `yourname` with your actual username):
   ```bash
   usermod -aG sudo yourname
   ```

3. **Exit and log back in:**
   ```bash
   exit
   ```
   Then log out completely and log back in.

4. **Test again:**
   ```bash
   sudo whoami
   ```
   Should now show: `root` 

---

### Step 1: Update Your System

This ensures you have the latest software:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y software-properties-common curl git wget build-essential
```

**What this does:** Downloads the latest package information and updates your system.

**Time:** 5-10 minutes

---

### Step 2: Set Up Swap Space (Prevents Crashes)

**What is swap?** Virtual memory that prevents your computer from freezing during compilation.

**First, check if you already have swap:**
```bash
sudo swapon --show
free -h
```

**If you see swap already (≥4GB):** Skip to Step 3.

**If no swap or swap is small, create 8GB swap:**

```bash
# Turn off existing swap if any
sudo swapoff /swapfile 2>/dev/null || true

# Remove old swap file
sudo rm -f /swapfile

# Create new 8GB swap
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make it permanent
grep -q '/swapfile' /etc/fstab || echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verify
free -h
```

You should see 8GB under "Swap" 

---

### Step 3: Install ALL Dependencies

This installs every library and tool needed. **Copy the entire block** and paste into terminal:

```bash
sudo apt install -y \
build-essential flex bison gawk tcl-dev tk-dev \
libx11-dev libx11-xcb-dev libxrender-dev libxpm-dev \
libncurses-dev libncurses5-dev blt freeglut3-dev \
mesa-common-dev libgl1-mesa-dev libglu1-mesa-dev \
libgtk-3-dev libgtk2.0-dev libcairo2 libcairo2-dev \
libxaw7 libxaw7-dev fontconfig libxft-dev libxmu6 libxmu-dev libxext-dev \
libreadline-dev libtool autoconf automake gettext \
python3 python3-pip python3-dev python3-venv python3-virtualenv \
cmake gfortran g++ gcc m4 tcsh csh ruby ruby-dev \
libboost-all-dev libboost-system-dev libboost-filesystem-dev \
libboost-thread-dev libboost-program-options-dev \
swig git-core \
libfftw3-dev libsuitesparse-dev libblas-dev liblapack-dev \
libglib2.0-dev \
libexpat1-dev libpng-dev zlib1g-dev \
qt5-default qtbase5-dev qttools5-dev qttools5-dev-tools \
libqt5widgets5 libqt5gui5 libqt5core5a \
libqt5multimedia5-plugins libqt5multimediawidgets5 qtmultimedia5-dev \
libqt5svg5-dev libqt5xmlpatterns5-dev libqt5designer5 \
libqt5x11extras5-dev \
libgit2-dev \
gperf clang libc++-dev libc++abi-dev \
libfl-dev pkg-config \
xterm xdot graphviz \
libeigen3-dev \
libxcb1-dev libxcb-render0-dev libxcb-shape0-dev libxcb-xfixes0-dev \
libbz2-dev liblzma-dev
```

**What this does:** Installs compilers, libraries, and Qt (for graphical tools).


---

### Step 4: Create Working Directories

Create folders where you'll build tools and design chips:

```bash
mkdir -p ~/sky130repo
mkdir -p ~/tools
mkdir -p ~/openpdk
mkdir -p ~/workingdir/{layout,schematic,verilog,simulations}
```

**What this creates:**
- `~/sky130repo/` - Source code for tools
- `~/tools/` - Where compiled tools will live
- `~/openpdk/` - Where the SkyWater PDK will be installed
- `~/workingdir/` - Where you'll design your chips

---

## Install All Tools

Now we'll install each tool one by one. **Important:** Install in this exact order!

---

### Tool 1: Magic (Layout Editor)

**What it does:** Lets you draw the physical layout of your chip.

```bash
cd ~/sky130repo

# Download Magic
git clone https://github.com/RTimothyEdwards/magic magic-git
cd magic-git

# Fix a common issue with readline
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

**Expected output:** Should show Magic version number 

---

### Tool 2: Xschem (Schematic Editor)

**What it does:** Draw circuit schematics (the "blueprint" of your circuit).

```bash
cd ~/sky130repo

# Download Xschem
git clone https://github.com/StefanSchippers/xschem.git xschem-git
cd xschem-git

# Build and install
./configure --prefix=$HOME/tools/xschem
make -j$(nproc)
make install

# Verify
$HOME/tools/xschem/bin/xschem --version
```

**Expected output:** Should show Xschem version 

**Time:** 3-5 minutes

---

### Tool 3: Ngspice (Circuit Simulator)

**What it does:** Simulates how your circuit behaves electrically.

```bash
cd ~/sky130repo

# Download Ngspice 45.2
wget https://sourceforge.net/projects/ngspice/files/ng-spice-rework/45.2/ngspice-45.2.tar.gz
tar -xzf ngspice-45.2.tar.gz
cd ngspice-45.2

# Build and install
./configure --prefix=$HOME/tools/ngspice \
    --with-x \
    --enable-xspice \
    --enable-cider \
    --with-readline=yes \
    --enable-openmp \
    --disable-debug

make -j$(nproc)
make install

# Verify
$HOME/tools/ngspice/bin/ngspice --version
```

**Expected output:** Should show Ngspice version 


---

### Tool 4: Netgen (LVS Tool)

**What it does:** Verifies that your layout matches your schematic (Layout vs Schematic check).

```bash
cd ~/sky130repo

# Download Netgen
git clone git://opencircuitdesign.com/netgen netgen-git
cd netgen-git

# Build and install
./configure --prefix=$HOME/tools/netgen
make -j$(nproc)
make install

# Verify
$HOME/tools/netgen/bin/netgen --version
```

**Expected output:** Should show Netgen version 


---

### Tool 5: KLayout (Layout Viewer)

**What it does:** View and edit chip layouts (alternative to Magic).

**First, install Qt5 packages for the GUI:**

```bash
sudo apt update
sudo apt install -y \
    qt5-default \
    qttools5-dev \
    qttools5-dev-tools \
    libqt5widgets5 \
    libqt5gui5 \
    libqt5core5a \
    libqt5multimedia5 \
    libqt5multimediawidgets5 \
    qtmultimedia5-dev \
    libqt5svg5-dev \
    libqt5x11extras5-dev
```

**Now build KLayout:**

```bash
cd ~/sky130repo

# Remove old builds if any
rm -rf klayout-git

# Download KLayout
git clone https://github.com/KLayout/klayout.git klayout-git
cd klayout-git

# Clean build directory
rm -rf build-release
mkdir build-release

# Build (takes 15-30 minutes)
./build.sh -j$(nproc) -prefix $HOME/tools/klayout

# Copy to tools directory
mkdir -p $HOME/tools/klayout/bin
cp -r bin-release/* $HOME/tools/klayout/bin/

# Verify
$HOME/tools/klayout/bin/klayout -v
```

**Expected output:** Should show KLayout version 


---

### Tool 6: Icarus Verilog (Digital Simulator)

**What it does:** Simulates digital circuits written in Verilog.

```bash
cd ~/sky130repo

# Download Icarus Verilog
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

**Expected output:** Should show Icarus Verilog version 

---

### Tool 7: GTKWave (Waveform Viewer)

**What it does:** View simulation waveforms (see how signals change over time).

**Easy method** (recommended):

```bash
sudo apt install -y gtkwave
```

**Verify:**
```bash
gtkwave --version
```

**Expected output:** Should show GTKWave version 

**Note:** You may see a warning about "canberra-gtk-module" - ignore it.


---

### Tool 8: Yosys (RTL Synthesis)

**What it does:** Converts Verilog code into a gate-level netlist.

```bash
cd ~/sky130repo

# Download Yosys
git clone https://github.com/YosysHQ/yosys.git yosys-git
cd yosys-git

# Configure
make config-gcc
echo "PREFIX := $HOME/tools/yosys" > Makefile.conf

# Update submodules
git submodule update --init

# Build (takes 5-15 minutes)
make -j$(nproc)
make install

# Verify
$HOME/tools/yosys/bin/yosys -V
```

**Expected output:** Should show Yosys version 


---

## 🎨 Install SkyWater PDK

**This is the most important step!** The PDK contains all the data about the 130nm fabrication process.

**⚠️ CRITICAL:** Magic must be installed first (you already did this in Tool 1).

### Download and Install PDK

```bash
cd ~/sky130repo

# Remove any old attempts
rm -rf open_pdks open_pdks-git

# Download PDK installer with submodules (critical!)
git clone --recurse-submodules https://github.com/RTimothyEdwards/open_pdks.git

cd open_pdks

# Verify download worked
ls -al
# You should see: README.md, configure, sky130/, etc.

# Initialize submodules (safety check)
git submodule update --init --recursive

# Make sure Magic is accessible
export PATH=$HOME/tools/magic/bin:$PATH
which magic
# Should show: /home/yourname/tools/magic/bin/magic

# Configure PDK for Sky130
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

**Expected output:** Should show `libs.ref` and `libs.tech` directories 


---

## ⚙️ Configure Your Environment

**This is critical!** Without this step, your tools won't find each other or the PDK.

### Add Environment Variables

Copy this **entire block** into your terminal:

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

# Apply changes immediately
source ~/.bashrc
```

### Verify Environment

```bash
# Check PDK variables
echo $PDK_ROOT
echo $SKY130A

# Both should show paths to your PDK installation
```

---

### Set Up Magic Configuration

```bash
# Copy Magic configuration
cp $SKY130A/libs.tech/magic/sky130A.magicrc ~/workingdir/layout/.magicrc
```

---

### Set Up Xschem Configuration

```bash
# Copy Xschem configuration
cp $SKY130A/libs.tech/xschem/xschemrc ~/workingdir/schematic/xschemrc

# Add Sky130 library paths
cat >> ~/workingdir/schematic/xschemrc << 'EOF'
set XSCHEM_LIBRARY_PATH $env(SKY130A)/libs.tech/xschem
append XSCHEM_LIBRARY_PATH :$env(HOME)/tools/xschem/share/xschem/xschem_library
EOF
```

---

## Verify Everything Works

Test each tool to make sure it's working properly:

### 1. Test Magic

```bash
cd ~/workingdir/layout
magic
```

**In the Magic console, type:**
```
tech
```

**Expected output:** Should show `sky130A` 

**To quit Magic:**
```
quit
```

---

### 2. Test Xschem

```bash
cd ~/workingdir/schematic
xschem
```

**In Xschem:**
- Press `Shift + I` (capital I)
- You should see a library browser
- Navigate to `sky130_fd_pr` folder
- You should see transistors and other Sky130 devices 

**To quit:** File → Quit

---

### 3. Test Ngspice

```bash
ngspice
```

**In ngspice, type:**
```
version
quit
```

**Expected output:** Should show Ngspice version 

---

### 4. Test All Tools

```bash
# Quick test of all tools
magic --version
xschem --version
ngspice --version
netgen --version
klayout -v
iverilog -V
gtkwave --version
yosys -V
```

**All should show version numbers without errors** 

---

## 🎓 Quick Command Reference

| Task | Command |
|------|---------|
| Open Magic | `cd ~/workingdir/layout && magic` |
| Open Xschem | `cd ~/workingdir/schematic && xschem` |
| Run Ngspice | `ngspice` |
| Open KLayout | `klayout` |
| View Waveforms | `gtkwave` |
| Check tool version | `toolname --version` |
| Reload environment | `source ~/.bashrc` |

---

## 🐛 Common Problems & Solutions

### Problem 1: "Command not found" after installation

**Solution:**
```bash
source ~/.bashrc
# OR close terminal and open a new one
```

---

### Problem 2: Magic says "technology not found"

**Solution:**
```bash
# Make sure you're in the layout directory
cd ~/workingdir/layout

# Check .magicrc exists
ls -la .magicrc

# If missing, copy it again
cp $SKY130A/libs.tech/magic/sky130A.magicrc ~/.magicrc
```

---

### Problem 3: Xschem doesn't show Sky130 devices

**Solution:**
```bash
cd ~/workingdir/schematic

# Make sure environment variable is set
echo $SKY130A

# If empty, reload environment
source ~/.bashrc

# Make sure xschemrc has correct paths
cat xschemrc | grep SKY130A
```

---

### Problem 4: KLayout won't build

**Solution:**
```bash
# Install missing Qt packages
sudo apt install -y \
    qt5-default \
    qttools5-dev \
    libqt5widgets5 \
    libqt5multimedia5 \
    qtmultimedia5-dev

# Clean and rebuild
cd ~/sky130repo/klayout-git
rm -rf build-release
./build.sh -j$(nproc) -prefix $HOME/tools/klayout
```

---

### Problem 5: PDK installation fails with "need magic"

**Solution:**
```bash
# Make sure Magic is in PATH
export PATH=$HOME/tools/magic/bin:$PATH

# Verify Magic works
magic --version

# Try PDK installation again
cd ~/sky130repo/open_pdks
./configure --enable-sky130-pdk --prefix=$HOME/openpdk
make -j$(nproc)
make install
```

---

### Problem 6: Swap file errors

**Solution:**
```bash
# Check current swap
sudo swapon --show

# If swap exists and works, skip swap creation
# If you need to recreate:
sudo swapoff /swapfile
sudo rm /swapfile
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

## 🎉 Next Steps - Start Designing!

Congratulations! You now have a complete VLSI design environment. Here's what you can do:

### 1. Draw Your First Schematic

```bash
cd ~/workingdir/schematic
xschem
```

- Press `Shift + I` to open component library
- Navigate to `sky130_fd_pr` → `nfet_01v8.sym`
- Click to place a transistor
- Draw a simple circuit

### 2. Create Your First Layout

```bash
cd ~/workingdir/layout
magic
```

- Use Sky130 layers to draw shapes
- Save your layout: `save mylayout`

### 3. Run Your First Simulation

Create a simple testbench in Xschem, then:
- Tools → Netlist
- Tools → Simulate (runs Ngspice)
- View results in Xschem or GTKWave

### 4. Try Digital Design

```bash
cd ~/workingdir/verilog
```

Create a file `test.v`:
```verilog
module test;
  initial begin
    $display("Hello, SkyWater PDK!");
    $finish;
  end
endmodule
```

Simulate it:
```bash
iverilog -o test test.v
vvp test
```

---
