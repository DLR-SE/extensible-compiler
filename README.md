# extensible-compiler

A lightweight tool for generating a C/C++ compiler for a RISC-V extension.

## Overview
Given a description of a RISC-V extension this tool patches a LLVM toolchain with support for the additional instructions. It is comprised of several submodules:
1. `RISCV-CoreDSL-Extensions` - CoreDSL descriptions of standard and custom extensions
1. `CoreDSL2TableGen` - generates patches for LLVM code from CoreDSL
1. `llvm` - current release of LLVM toolchain prepared for patching

The standard workflow is preparing the CoreDSL description of the extension, generating and applying the LLVM patches, then building the compiler toolchain.

This README provides a sketch of building the patched LLVM toolchain on an up-to-date Linux host environment. Note that LLVM supports a range of host environments and is highly configurable: https://llvm.org/docs/GettingStarted.html is a useful starting point.

## Clone the repository including submodules

Cloning via SSH is straightforward:

    git clone git@github.com:DLR-SE/extensible-compiler.git --recurse-submodules

Alternatively if SSH is blocked (e.g. by a firewall) then cloning via HTTPS needs the submodule paths substituted:

    git config --global url.https://github.com/.insteadOf git@github.com:
    git clone https://github.com/DLR-SE/extensible-compiler.git --recurse-submodules

## Add the coredsl and metadata for the extension

See CoreDSL2TableGen/README.md for details on adding the extension and patching the `llvm` repository

## Building the compiler

As preparation: the host needs a current system compiler (GCC or clang) and cmake package installed (useful diagnostics are given if suitable versions aren't located). It is also preferable to export the built compiler to a local location rather than attempting to replace the system compiler - in the following  example `$HOME/tools` is used:

Starting in the `llvm` directory

    cmake llvm -G "Ninja" -B build -DLLVM_USE_LINKER=lld -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD='X86;RISCV' -DLLVM_ENABLE_PROJECTS='clang;lld' -DCMAKE_INSTALL_PREFIX=$HOME/llvm -DLLVM_OPTIMIZED_TABLEGEN=1
    ninja -C build
    ninja -C build install
 
## Using the compiler

Prefix the path with the exported toolchain bin directory

    export PATH=~/llvm/bin:$PATH

Custom extensions are not included by default and must be explicitly enabled when compiling by adding them to the architecture list prefixed by the letter 'x' (experimental). 

    clang -target riscv32 -march=rv32ixrbnn -c ../RISCV-CoreDSL-Extensions/bosch_NN.c

The test file used in this example also demonstrates how a regression test case can be added for an extension, using the llvm-lit and FileCheck tools to automate its execution and evaluation. For this it needs to be copied to a suitable path within the test hierarchy:

    mkdir -p clang/test/CodeGen/riscv-s4e
    cp ../RISCV-CoreDSL-Extensions/bosch_NN.c clang/test/CodeGen/riscv-s4e 

Then to run just this one test case

    env LIT_FILTER=bosch_nn.c llvm-lit -v clang/test/CodeGen

or to run the complete regression test set including this test case:

    ninja -C build check

Documentation for this is under https://llvm.org/docs/TestingGuide.html
