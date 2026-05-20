# The Complete Klutz's Guide for Packaing for Debian

## The Why

## The How

## The Walkthrough

### [wasmparser](https://github.com/bytecodealliance/wasm-tools) in [wasmi](https://github.com/wasmi-labs/wasmi)

Zellij depends on wasmi, which uses [wasmparser](https://github.com/bytecodealliance/wasm-tools). Here, I need to update the wasmparser dependency from `0.228` to `0.239`[^1]. 
First, I cloned the repository locally:

``git clone https://github.com/wasmi-labs/wasmi.git``
``cd wasmi``

The cloned repo is probabaly on main, but I want to work on the latest stable release. At the time of writing, that's v1.0.9 released Feb 9th, 2026. I'll also create a dedicated branch.

``git checkout v1.0.9`` 
``git checkout -b bump-wasmparser-0.239``


In a Rust project, Cargo.toml is where project metadata and dependencies are declared.
So, I look which Cargo.toml files reference wasmparser (and to which versions):

``grep -rn "wasmparser" --include="*.toml" . ``

> ./crates/wasmi/Cargo.toml:25:wasmparser = { workspace = true, features = ["validate", "features"] }  
> ./crates/wasmi/Cargo.toml:42:    "wasmparser/std",  
> ./crates/wasmi/Cargo.toml:47:    "wasmparser/hash-collections",  
> ./crates/wasmi/Cargo.toml:51:    "wasmparser/prefer-btree-collections",  
> ./crates/wasmi/Cargo.toml:54:simd = ["wasmi_core/simd", "wasmi_ir/simd", "wasmparser/simd"]  
> ./Cargo.toml:48:wasmparser = { version = "0.228.0", default-features = false }  

The output tells me that the project uses wasmparser v.0.228.0, as expected. I also see the workspace uses the validate and features options, as well as others like std and simd. 

I also want to look at the Cargo.lock file, which mentions the exact resolved version of every dependency:

`` grep -rn "wasmparser" --include="*.lock" .``

> ./Cargo.lock:1671: "wasmparser 0.228.0",  
> ./Cargo.lock:1681: "wasmparser 0.232.0",  
> ./Cargo.lock:1691: "wasmparser 0.240.0",    
 ./Cargo.lock:1716: "wasmparser-nostd",  
 ./Cargo.lock:1732: "wasmparser 0.228.0",  
 ./Cargo.lock:1746: "wasmparser 0.228.0",  
 ./Cargo.lock:1892:name = "wasmparser"  
 ./Cargo.lock:1903:name = "wasmparser"  
 ./Cargo.lock:1913:name = "wasmparser"
 ./Cargo.lock:1926:name = "wasmparser-nostd"
 ./Cargo.lock:1942: "wasmparser 0.228.0",
 ./Cargo.lock:1953: "wasmparser 0.240.0",
 ./Cargo.lock:1984: "wasmparser 0.240.0",
 ./Cargo.lock:2017: "wasmparser 0.240.0",
 ./Cargo.lock:2042: "wasmparser 0.240.0",
 ./Cargo.lock:2137: "wasmparser 0.240.0",
 ./Cargo.lock:2250: "wasmparser 0.240.0",
</blockquote>

Wow, that's quite the picture. wasmi uses 3 different version of wasmparser, possibly for other dependencies that need specific versions. There's also wasmparser-nostd, which we may have to look at later.

Why the different versions? I can spend hours digging the code, or ask the maintainers. So I opened [an issue with some questions](https://github.com/wasmi-labs/wasmi/issues/1858).

Next update is when they reply, hopefully. 
- May 20th 2026 

[^1]: I'm not sure why that's needed, as I inherited this part of the project. The Changelog files did not provide clear answers. 
