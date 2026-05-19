# The Complete Klutz's Guide for Packaing for Debian

## The Why

## The How

## The Walkthrough

### [wasmparser](https://github.com/bytecodealliance/wasm-tools) in [wasmi](https://github.com/wasmi-labs/wasmi)

Zellij depends on wasmi, which uses [wasmparser](https://github.com/bytecodealliance/wasm-tools). Here, I need to update the wasmparser dependency from `0.228` to `0.239`[^1]. 
First, I cloned the repository locally

``git clone https://github.com/wasmi-labs/wasmi.git
cd wasmi``


[^1]: I'm not sure why that's needed, as I inherited this part of the project. The Changelog files did not provide clear answers. 
