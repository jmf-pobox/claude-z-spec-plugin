---
description: Install and configure fuzz and probcli dependencies
argument-hint: "[check|fuzz|probcli|all]"
---

# Setup Z Specification Tools

You are helping the user install and configure the tools needed for Z specification development.

## Input

Arguments: $ARGUMENTS

Parse as:
- `check` - Check what's installed and report status
- `fuzz` - Install fuzz type-checker
- `probcli` - Install ProB command-line interface
- `tex` - Copy TeX files (fuzz.sty, *.mf) to project docs/ directory
- `all` - Install everything including TeX files
- (no argument) - Same as `check`

## Process

### 1. Detect Platform

```bash
uname -s  # Darwin, Linux, etc.
uname -m  # arm64, x86_64, etc.
```

### 2. Check Current Status

Always start by checking what's already installed:

```bash
# Check fuzz
which fuzz && fuzz -version

# Check probcli
which probcli || test -x "$HOME/Applications/ProB/probcli" && echo "probcli found"

# Check fuzz.sty in TeX path
kpsewhich fuzz.sty

# Check Tcl/Tk (needed for probcli on some systems)
which wish || brew list tcl-tk 2>/dev/null
```

Report status clearly:
```
## Current Status

| Tool | Status |
|------|--------|
| fuzz | ✓ Installed (version X) |
| fuzz.sty | ✓ Found in TeX path |
| probcli | ✗ Not found |
| Tcl/Tk | ✓ Available |
```

### 3. Install fuzz

fuzz is the Z type-checker. It must be compiled from source.

#### Prerequisites

**macOS:**
```bash
# Xcode command line tools (for gcc/make)
xcode-select --install

# TeX distribution (for fuzz.sty installation)
# User should have MacTeX or BasicTeX installed
```

**Linux (Debian/Ubuntu):**
```bash
sudo apt-get install build-essential texlive-base
```

#### Installation Steps

```bash
# Clone fuzz repository
cd ~/Applications  # or user's preferred location
git clone https://github.com/Spivoxity/fuzz.git
cd fuzz

# Build
make

# Install fuzz.sty to TeX path (may need sudo)
sudo make install

# Verify
fuzz -version
kpsewhich fuzz.sty
```

#### Add to PATH

If `fuzz` isn't in PATH after building:

```bash
# Add to shell profile (~/.zshrc or ~/.bashrc)
export PATH="$HOME/Applications/fuzz:$PATH"
```

#### Common Issues

**"fuzz.sty not found"**: Run `sudo make install` in the fuzz directory, then `sudo texhash`.

**"make: gcc: command not found"**: Install Xcode command line tools: `xcode-select --install`

### 4. Install probcli

probcli is the ProB command-line interface for animating and model-checking Z specifications.

#### macOS Installation

**Option A: Standalone CLI (Recommended)**

```bash
# Create directory
mkdir -p ~/Applications/ProB
cd ~/Applications/ProB

# Download latest release
# Check https://prob.hhu.de/w/index.php/Download for current version
curl -L -o probcli.zip "https://prob.hhu.de/downloads/prob2-latest/prob-macOS.zip"

# Or for Apple Silicon specifically:
# curl -L -o probcli.zip "https://prob.hhu.de/downloads/prob2-latest/prob-macOS-aarch64.zip"

unzip probcli.zip
chmod +x probcli

# Verify
./probcli -version
```

**Option B: Full ProB Installation**

Download the full ProB application from https://prob.hhu.de/w/index.php/Download which includes the GUI and all dependencies.

#### Tcl/Tk Dependency

probcli may require Tcl/Tk libraries even in CLI mode. On macOS:

```bash
# Install via Homebrew
brew install tcl-tk

# Add to shell profile if needed
export PATH="/opt/homebrew/opt/tcl-tk/bin:$PATH"
export LDFLAGS="-L/opt/homebrew/opt/tcl-tk/lib"
export CPPFLAGS="-I/opt/homebrew/opt/tcl-tk/include"
```

#### Add to PATH

```bash
# Add to shell profile (~/.zshrc or ~/.bashrc)
export PROBCLI="$HOME/Applications/ProB/probcli"
export PATH="$HOME/Applications/ProB:$PATH"
```

Or create a symlink:
```bash
sudo ln -s ~/Applications/ProB/probcli /usr/local/bin/probcli
```

#### Linux Installation

```bash
# Download
mkdir -p ~/Applications/ProB
cd ~/Applications/ProB
curl -L -o probcli.zip "https://prob.hhu.de/downloads/prob2-latest/prob-linux64.zip"
unzip probcli.zip
chmod +x probcli

# Install Tcl/Tk if needed
sudo apt-get install tcl tk

# Verify
./probcli -version
```

#### Common Issues

**"dyld: Library not loaded: libtcl"**: Install Tcl/Tk via Homebrew: `brew install tcl-tk`

**"probcli: cannot execute binary file"**: Wrong architecture. Download the correct version (Intel vs Apple Silicon).

**"Error: PROB_HOME not set"**: Set environment variable:
```bash
export PROB_HOME="$HOME/Applications/ProB"
```

**Quarantine on macOS**: If macOS blocks the binary:
```bash
xattr -d com.apple.quarantine ~/Applications/ProB/probcli
```

### 5. Copy TeX Files to Project

Z specifications require `fuzz.sty` and Metafont files. Rather than configure system TeX paths, copy them directly to the project's `docs/` directory.

#### Check Local Installation First

```bash
# Standard install location from `make install`
LOCAL_STY="/usr/local/share/texmf/tex/latex/fuzz.sty"
LOCAL_MF="/usr/local/share/texmf/fonts/source/public/oxsz"

if [ -f "$LOCAL_STY" ]; then
    echo "Found fuzz.sty at $LOCAL_STY"
fi

if [ -d "$LOCAL_MF" ]; then
    echo "Found Metafont files at $LOCAL_MF"
fi
```

#### Copy from Local Install

If files exist locally:

```bash
# Create docs directory if needed
mkdir -p docs

# Copy fuzz.sty
cp /usr/local/share/texmf/tex/latex/fuzz.sty docs/

# Copy Metafont files (needed for PDF compilation with Z symbols)
cp /usr/local/share/texmf/fonts/source/public/oxsz/*.mf docs/
```

#### Fallback: Download from GitHub

If local files not found, download from the fuzz repository:

```bash
mkdir -p docs
cd docs

# Required: fuzz.sty
curl -LO "https://raw.githubusercontent.com/Spivoxity/fuzz/master/tex/fuzz.sty"

# Metafont files for Z symbols
for mf in oxsz.mf oxsz10.mf oxsz5.mf oxsz6.mf oxsz7.mf oxsz8.mf oxsz9.mf zarrow.mf zletter.mf zsymbol.mf; do
    curl -LO "https://raw.githubusercontent.com/Spivoxity/fuzz/master/tex/$mf"
done
```

#### Verify TeX Files

```bash
ls -la docs/fuzz.sty docs/*.mf
```

Expected files:
- `fuzz.sty` - LaTeX style defining Z notation commands
- `oxsz*.mf` - Metafont sources for Z symbol font
- `zarrow.mf`, `zletter.mf`, `zsymbol.mf` - Additional symbol definitions

### 6. Verify Installation

After installation, verify everything works:

```bash
# Test fuzz
echo '\begin{zed}[X]\end{zed}' > /tmp/test.tex
fuzz -t /tmp/test.tex

# Test probcli
probcli -version
```

Create a simple test spec and run both tools:

```bash
cat > /tmp/test_spec.tex << 'EOF'
\documentclass{article}
\usepackage{fuzz}
\begin{document}
\begin{zed}
[ID]
\end{zed}
\begin{zed}
ZBOOL ::= ztrue | zfalse
\end{zed}
\begin{schema}{State}
count : \nat
\where
count \leq 100
\end{schema}
\begin{schema}{Init}
State'
\where
count' = 0
\end{schema}
\end{document}
EOF

fuzz -t /tmp/test_spec.tex && echo "fuzz: OK"
probcli /tmp/test_spec.tex -init && echo "probcli: OK"
```

### 7. Report Results

Summarize what was done and current status:

```
## Setup Complete

| Tool | Status | Location |
|------|--------|----------|
| fuzz | ✓ Installed | ~/Applications/fuzz/fuzz |
| fuzz.sty | ✓ Installed | /usr/local/texlive/.../fuzz.sty |
| probcli | ✓ Installed | ~/Applications/ProB/probcli |

## Shell Configuration

Add to ~/.zshrc:
```bash
export PATH="$HOME/Applications/fuzz:$HOME/Applications/ProB:$PATH"
export PROBCLI="$HOME/Applications/ProB/probcli"
```

Run `source ~/.zshrc` or restart your terminal.
```

## Interactive Guidance

If installation fails or user needs help:

1. **Ask about their environment**: macOS version, Intel vs Apple Silicon, existing TeX installation
2. **Diagnose specific errors**: Parse error messages and suggest fixes
3. **Offer alternatives**: If probcli won't install, fuzz alone is still useful for type-checking
4. **Test incrementally**: Verify each step before proceeding

