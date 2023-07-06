# extensible-compiler

A lightweight tool for generating a C/C++ compiler for a RISC-V extension.

## Overview
Given a description of a RISC-V extension this tool patches a LLVM toolchain with support for the additional instructions. It is comprised of several submodules:
1. `RISCV-CoreDSL-Extensions` - CoreDSL descriptions of public standard and custom extensions
1. `RISCV-CoreDSL-Partner-Extensions` - CoreDSL descriptions of Scale4Edge-partner extensions 
1. `CoreDSL2TableGen` - generates patches for LLVM code from CoreDSL
1. `llvm` - current release of LLVM toolchain prepared for patching

The standard workflow is preparing the CoreDSL description of the extension, generating and applying the LLVM patches, then building the compiler toolchain. This README provides a sketch of building the patched LLVM toolchain on an up-to-date Linux host environment. Note that LLVM supports a range of host environments and is highly configurable: https://llvm.org/docs/GettingStarted.html is a useful starting point.

## Clone the repository

Users who do not belong to the Scale4Edge project should jump to the following "Anonymous public access" sub-section.

### Cloning by Scale4Edge project members

Before being able to clone the partner-only material you will need access to be granted in GitHub and to authenticate yourself during the cloning.

Cloning via SSH is straightforward:

    git clone git@github.com:DLR-SE/extensible-compiler.git --recurse-submodules

Alternatively if SSH is blocked (e.g. by a firewall) then cloning via HTTPS needs the submodule paths substituted:

    git config --global url.https://github.com/.insteadOf git@github.com:
    git clone https://github.com/DLR-SE/extensible-compiler.git --recurse-submodules

### Cloning by anonymous public access

To avoid errors you need to not clone the partner-only repository.

Cloning via SSH is straightforward:

    git -c submodule.RISCV-CoreDSL-Partner-Extensions.update=none clone git@github.com:DLR-SE/extensible-compiler.git --recurse-submodules

Alternatively if SSH is blocked (e.g. by a firewall) then cloning via HTTPS needs the submodule paths substituted:

    git config --global url.https://github.com/.insteadOf git@github.com:
    git -c submodule.RISCV-CoreDSL-Partner-Extensions.update=none clone https://github.com/DLR-SE/extensible-compiler.git --recurse-submodules

Configuring the partner sub-module as non-cloned avoids the need to specify this again on subsequent git commands:

    git -C extensible-compiler config submodule.RISCV-CoreDSL-Partner-Extensions.update none 

## Add the coredsl and metadata for the extension

See CoreDSL2TableGen/README.md for details on generating support for an extension and patching the `llvm` repository. The following sections are written for the sample S4E_MAC extension; you will need to substitute names and paths appropriately for other extensions.

## Building the compiler

As preparation: the host needs a current system compiler (GCC or clang) and cmake package installed (useful diagnostics are given if suitable versions aren't located). It is also preferable to export the built compiler to a local location rather than attempting to replace the system compiler - in the following  example `$HOME/tools` is used:

Starting in the `llvm` directory

    cmake llvm -G "Ninja" -B build -DLLVM_USE_LINKER=lld -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD='RISCV' -DLLVM_ENABLE_PROJECTS='clang;lld' -DCMAKE_INSTALL_PREFIX=$HOME/llvm -DLLVM_EXTERNAL_LIT=build/bin/llvm-lit
    ninja -C build
    ninja -C build install
 
## Using the compiler

Prefix the path with the exported toolchain bin directory

    export PATH=~/llvm/bin:$PATH

Experimental extensions are not enabled by default and must be explicitly enabled when compiling by adding them to the architecture list prefixed by the letter 'x'. 

    clang -target riscv32 -march=rv32ixs4emac -c ../RISCV-CoreDSL-Extensions/s4e-mac.test.c

The test file used in this example also demonstrates how a regression test case can be added for an extension, using the llvm-lit and FileCheck tools to automate its execution and evaluation. For this it needs to be copied to a suitable path within the test hierarchy:

    mkdir -p clang/test/CodeGen/riscv-s4e
    cp ../RISCV-CoreDSL-Extensions/s4e-mac.test.c clang/test/CodeGen/riscv-s4e 

Then to run just this one test case

    env LIT_FILTER=s4e-mac.test.c build/bin/llvm-lit -v clang/test/CodeGen

or to run the complete regression test set including this test case:

    ninja -C build check

Documentation for this is under https://llvm.org/docs/TestingGuide.html
