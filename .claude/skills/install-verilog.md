---
name: install-verilog
description: Install Verilog development tools (Verilator, GTKWave, Bison, Flex, SystemC) on Apple Silicon Mac following k0nze.dev guide
user_invocable: true
---

# Install Verilog on Apple Silicon

Guide based on: https://k0nze.dev/posts/verilog-apple-silicon/

Run each step sequentially. Ask the user before proceeding with steps that require `sudo`.

## Step 1: Xcode Command Line Tools

```bash
xcode-select --install
```

## Step 2: Homebrew (skip if already installed)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Ensure Homebrew is in PATH (`/opt/homebrew/bin`).

## Step 3: Core dependencies

```bash
brew install cmake ninja wget git
```

## Step 4: GTKWave

```bash
brew install --HEAD randomplum/gtkwave/gtkwave
```

## Step 5: Build Bison 3.7.91

```bash
sudo mkdir -p /opt/bison/bison-3.7.91
cd /tmp
wget http://alpha.gnu.org/gnu/bison/bison-3.7.91.tar.xz
tar -xvf bison-3.7.91.tar.xz
cd bison-3.7.91
./configure --prefix=/opt/bison/bison-3.7.91 CFLAGS="-isysroot $(xcrun -show-sdk-path)" CXXFLAGS="-isysroot $(xcrun -show-sdk-path)"
make
sudo make install
```

Add to `~/.zshrc`:
```bash
export CFLAGS="-I/opt/bison/bison-3.7.91/share/include ${CFLAGS}"
export CXXFLAGS="-I/opt/bison/bison-3.7.91/share/include ${CXXFLAGS}"
export LDFLAGS="-L/opt/bison/bison-3.7.91/lib ${LDFLAGS}"
export PATH="/opt/bison/bison-3.7.91/bin:${PATH}"
```

## Step 6: Build Flex 2.6.4

```bash
sudo mkdir -p /opt/flex/flex-2.6.4
cd /tmp
wget https://github.com/westes/flex/releases/download/v2.6.4/flex-2.6.4.tar.gz
tar -xvf flex-2.6.4.tar.gz
cd flex-2.6.4
./configure --prefix=/opt/flex/flex-2.6.4 CFLAGS="-isysroot $(xcrun -show-sdk-path)" CXXFLAGS="-isysroot $(xcrun -show-sdk-path)"
make
sudo make install
```

Add to `~/.zshrc`:
```bash
export CFLAGS="-I/opt/flex/flex-2.6.4/include ${CFLAGS}"
export CXXFLAGS="-I/opt/flex/flex-2.6.4/include ${CXXFLAGS}"
export LDFLAGS="-L/opt/flex/flex-2.6.4/lib ${LDFLAGS}"
export PATH="/opt/flex/flex-2.6.4/bin:${PATH}"
```

## Step 7: Install SystemC

Ask the user where to clone SystemC (e.g. `~/tools/systemc`), then:

```bash
export SYSTEMC_HOME="<chosen-path>"
git clone https://github.com/accellera-official/systemc "$SYSTEMC_HOME"
cd "$SYSTEMC_HOME"
git checkout 2.3.3
mkdir build && cd build
cmake -GNinja -DENABLE_PTHREADS=ON -DENABLE_PHASE_CALLBACKS_TRACING=OFF -DCMAKE_CXX_STANDARD=17 ..
ninja
ninja install
```

Add `export SYSTEMC_HOME="<chosen-path>"` to `~/.zshrc`.

## Step 8: Install Verilator

Ask the user where to clone Verilator (e.g. `~/tools/verilator`), then:

```bash
export VERILATOR_ROOT="<chosen-path>"
git clone https://github.com/verilator/verilator "$VERILATOR_ROOT"
cd "$VERILATOR_ROOT"
git checkout v4.224
autoconf
./configure
make
```

Add to `~/.zshrc`:
```bash
export VERILATOR_ROOT="<chosen-path>"
export PATH="$VERILATOR_ROOT/bin:$PATH"
export VERILATOR_INC_DIR="$VERILATOR_ROOT/include"
```

## Step 9: Reload shell

```bash
source ~/.zshrc
```

## Step 10: Verify

```bash
verilator --version
gtkwave --version
```

## Notes
- Steps with `sudo` require user confirmation before execution.
- Ask the user for installation paths for SystemC and Verilator before proceeding.
- If any step fails, diagnose the error before continuing.
