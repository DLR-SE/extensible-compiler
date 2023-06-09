# extensible-compiler

A lightweight tool for generating a C/C++ compiler for a RISC-V extension.

## Overview
Given a description of a RISC-V extension this tool patches a LLVM toolchain with support for the additional instructions. It is comprised of several submodules:
1. riscv-coredsl-extensions - CoreDSL descriptions of standard and custom extensions
1. coredsl2tablegen - generates patches for LLVM code from CoreDSL
1. llvm - current release of LLVM toolchain prepared for patching

The standard workflow is preparing the CoreDSL description of the extension, generating and applying the LLVM patches, then building the compiler toolchain.

This README provides a sketch of building the patched LLVM toolchain on an up-to-date Linux host environment. Note that LLVM supports a range of host environments and is highly configurable: https://llvm.org/docs/GettingStarted.html is a useful starting point.

## Clone the repository including submodules

git clone https://github.com/DLR-SE/extensible-compiler.git --recurse-submodules

## Add the coredsl and metadata for the extension

See CoreDSL2TableGen/README.md for details on adding the extension and patching the `llvm` repository

## Building the compiler

As preparation: the host needs a current system compiler (GCC or clang) and cmake package installed (useful diagnostics are given if suitable versions aren't located). It is also preferable to export the built compiler to a local location rather than attempting to replace the system compiler - in the following  example `$HOME/tools` is used:

Starting in the `llvm` directory
    cmake llvm -G "Ninja" -B build -DLLVM_USE_LINKER=lld -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD='X86;RISCV' -DLLVM_ENABLE_PROJECTS='clang;lld' -DCMAKE_INSTALL_PREFIX=$HOME/llvm -DLLVM_OPTIMIZED_TABLEGEN=1
    ninja -C build
    ninja -C build install
