# The Complete Klutz's Guide for Packaing for Debian

## What is packaging?

*A Debian package is a collection of files that allow for applications or libraries to be distributed via the package management system. The aim of packaging is to allow the automation of installing, upgrading, configuring, and removing computer programs for Debian in a consistent manner.*[^1]

In other words, when software is packaged for an OS, it can be easily installed and automatically upgraded. It's more lightweight than containers and allows the user to install/remove/update using the OS management system - in Linux, that's `apt`.

*A package consists of one source package, and one or more binary packages. The Debian Policy specifies the standard format for a package, which all packages must follow. Source packages contain the upstream source distribution, configuration for the package build system, list of runtime dependencies and conflicting packages, a machine-readable description of copyright and license information, initial configuration for the software, and more.*[^1]

That's a mouthful if you haven't got the technical background. The package is "everything needed for the installation" - the files, the dependencies, instructions and settings - all bundled. We'll get more into this later, but for now, it's enough to know that packaging produces the source package (.dsc file) and binary packages (.deb).

## Why package for Debian?

To make an application more accessible and to increase user trust. More accessible, because obtaining the application would be faster and simpler. More trustworthy because official repositories are curated by the OS maintainers and must stick to strict guidelines[^1] - making it more secure and less likely that the installtion would fck things up. It also principally saves the user storage space.
Debian serves as a base for other popular systems including Ubuntu. Hence, software that's packaged for debian is compatible with many other Linux distributions. Once packaged for Debian, it's much easier to package for Mint and Ubuntu, for example.

## The How

The complete walkthrough depends on what you're packaging - the language it's written in, the direct dependencies and [whether those dependencies exist in Debian's repository](#list-the-dependency-tree). The steps listed below are specific to packaging [zellij](https://zellij.dev), a large project writted in Rust.

### The steps

1. Write software.
    :ballot_box_with_check: Done: we're packaging [zellij](https://zellij.dev/).
2. Check if the project is being [worked on](https://www.debian.org/devel/wnpp/being_packaged)[^2]. If it is, you can still help.
3. [List the dependency Tree](#list-the-dependency-tree). Our code depends on library
4. Package the dependencies - or
5. Bundle everything into a package
6. Publish the repository in the Debian unstable version 
7. Have it tested for bugs until the package passes testing.
8. Publish the repository in the Debian stable version.
6. Celebrate - optionally with a trip to [Italy](https://upload.wikimedia.org/wikipedia/commons/9/99/Collage_Roma.jpg)


## The Walkthrough

### 3. List the Dependency Tree



### [wasmparser](https://github.com/bytecodealliance/wasm-tools) in [wasmi](https://github.com/wasmi-labs/wasmi)

Zellij depends on wasmi, which uses [wasmparser](https://github.com/bytecodealliance/wasm-tools). Here, I need to update the wasmparser dependency from `0.228` to `0.239`[^3]. 
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
> ./Cargo.lock:1716: "wasmparser-nostd",    
> ./Cargo.lock:1732: "wasmparser 0.228.0",   
> ./Cargo.lock:1746: "wasmparser 0.228.0",  
> ./Cargo.lock:1892:name = "wasmparser"  
> ./Cargo.lock:1903:name = "wasmparser"  
> ./Cargo.lock:1913:name = "wasmparser"  
> ./Cargo.lock:1926:name = "wasmparser-nostd"  
> ./Cargo.lock:1942: "wasmparser 0.228.0",  
> ./Cargo.lock:1953: "wasmparser 0.240.0",  
> ./Cargo.lock:1984: "wasmparser 0.240.0",  
> ./Cargo.lock:2017: "wasmparser 0.240.0",  
> ./Cargo.lock:2042: "wasmparser 0.240.0",  
> ./Cargo.lock:2137: "wasmparser 0.240.0",  
> ./Cargo.lock:2250: "wasmparser 0.240.0",  


Wow, that's quite the picture. wasmi uses 3 different version of wasmparser, possibly for other dependencies that need specific versions. There's also wasmparser-nostd, which we may have to look at later.

Why the different versions? I can spend hours digging the code, or ask the maintainers. So I opened [an issue with some questions](https://github.com/wasmi-labs/wasmi/issues/1858).

Next update is when they reply, hopefully. 
- May 20th 2026 

[^1]: [Debian Intro to packaging](https://wiki.debian.org/Packaging/Intro)
[^2]: [Debian Mentors FAQ: How do I make my first package?](https://wiki.debian.org/DebianMentorsFaq#How_do_I_make_my_first_package.3F)
[^3]: This is a part of the project that I inherited, so I cannot fully explain the dependencies issues here.
