# 🐉 Rust GPU

[![Embark](https://img.shields.io/badge/embark-open%20source-blueviolet.svg)](https://embark.dev)
[![Embark](https://img.shields.io/badge/discord-ark-%237289da.svg?logo=discord)](https://discord.gg/dAuKfZS)
[![dependency status](https://deps.rs/repo/github/EmbarkStudios/rust-gpu/status.svg)](https://deps.rs/repo/github/EmbarkStudios/rust-gpu)
[![Build status](https://github.com/EmbarkStudios/rust-gpu/workflows/Continuous%20integration/badge.svg?branch=main)](https://github.com/EmbarkStudios/rust-gpu/actions)

This is a very early stage project to make Rust a first-class language and ecosystem for building GPU code 🚀🚧

### Current Status: v0.1

Compiling and running very simple shaders works, and a significant portion of [the core library](https://doc.rust-lang.org/core/index.html) also compiles.

However, many things aren't implemented yet: for example, loops and switches aren't supported yet! That means that while being technically usable, this project is far from being production-ready.

### Example

![Sky shader](docs/assets/sky.jpg)

```rust
#[spirv(entry = "fragment")]
pub fn main_fs(input: Input<Vec4>, mut output: Output<Vec4>) {
    let dir: Vec3 = input.load().truncate();

    let cs_pos = Vec4(dir.0, -dir.1, 1.0, 1.0);
    let ws_pos = {
        let p = clip_to_world.mul_vec4(cs_pos);
        p.truncate() / p.3
    };
    let dir = (ws_pos - eye_pos).normalize();
    
    // evaluate Preetham sky model
    let color = sky(dir, sun_pos);

    output.store(color.extend(0.0))
}
```

See [source](examples/example-shader/src/lib.rs) for full details.

## Background

Historically in games GPU programming has been done through writing either HLSL, or to a lesser extent GLSL. These are simple programming languages that have evolved along with rendering APIs over the years. However, as game engines have evolved, these languages have failed to provide mechanisms for dealing with large codebases, and have generally stayed behind the curve compared to other programming languages.

In part this is because it's a niche language for a niche market, and in part this has been because the industry as a whole has sunk quite a lot of time and effort into the status quo. While over-all better alternatives to both languages exist, none of them are in a place to replace HLSL or GLSL. Either because they are vendor locked, or because they don't support the traditional graphics pipeline. Examples of this include CUDA and OpenCL. And while attempts have been made to create language in this space, none of them have gained any notable traction in the gamedev community.

Our hope with this project is that we push the industry forward by bringing an existing, low-level, safe, and high performance language to the GPU; namely [Rust](https://rust-lang.org). And with it come some additional benefits that can't be overlooked: a package/module system that's one of the industry's best, built in safety against race-conditions or out of bounds memory access, a wide range of tools and utilities to improve programmer workflows, and many others!

### Why Embark?

At Embark, we've been building our own new game engine from the ground up in Rust. We have previous experience in-house developing the [RLSL](https://github.com/MaikKlein/rlsl) prototype, and we have a team of excellent rendering engineers that are familiar with the problems in current shading languages both from games, game engines and other industries. So, we believe we are uniquely positioned to attempt solving this problem. 

We want to streamline our own internal development with a single great language, build an open source graphics ecosystem and community, facilitate code-sharing between GPU and CPU, and most importantly: to enable our (future) users, and fellow developers, to more rapidly build great looking and engaging experiences.

If we do this project right, one wouldn't necessarily need an entire team of rendering engineers to build a good looking game, instead one would simply use a few of the existing open-source crates that provide the graphical effects needed to create the experience you're after. Instead of sharing and copy'n'pasting snippets of TAA code on forum posts, one could simply find and use the right crates from [crates.io](https://crates.io).

## Project scope

The scope of this overall project is quite broad, but is in multiple stages

- `rustc` compiler backend to generate [SPIR-V], plugging in via `-Z codegen-backend`. 
  - This is the same mechanism that [rustc_codegen_cranelift](https://github.com/bjorn3/rustc_codegen_cranelift) and [rustc_codegen_gcc](https://github.com/antoyo/rustc_codegen_gcc) use.
  - Currently only [SPIR-V] support is planned, [Vulkan](https://en.wikipedia.org/wiki/Vulkan_(API))'s open compiler target
  - Possible a future version could suppport [DXIL](https://github.com/microsoft/DirectXShaderCompiler/blob/master/docs/DXIL.rst) (the target for DirectX) or [WGSL](https://github.com/gpuweb/gpuweb/tree/main/wgsl) (the WebGPU shading language that's bijective with SPIR-V)
- Focus on Vulkan graphics shaders first, then after Vulkan compute shaders
- [Cargo](https://github.com/rust-lang/cargo/) and [crates.io](https://crates.io) support to develop and publish SPIR-V crates
- High-level render graph to take advantage of this, make it easy for users to develop and use rendering effects.

An in-depth exploration of our roadmap and milestones can be found [here](https://github.com/EmbarkStudios/rust-gpu/issues/47).

## Process

We use this repo as a monorepo for everything related to the project: crates, tools, shaders, examples, tests, and design documents. This way, we can use issues and PRs covering everything in the same place, cross-reference stuff within the repo, as well as with other GitHub repos such as [rspirv](https://github.com/gfx-rs/rspirv) and [Rust](https://github.com/rust-lang/rust) itself.

We meet weekly over a Discord call to discuss design and triage issues. Each meeting has an [issue](https://github.com/EmbarkStudios/rust-gpu/issues?q=label%3Ameeting+) with agenda, links and minutes.

We have a [#rust-gpu Discord channel](https://discord.gg/dAuKfZS) for fast discussion and collaboration.

## Backwards compatibility, breaking changes and deprecation

Right now because the project is in a very early state of development, we might introduce temporary changes as stop-gap measures, or implement features or APIs that might not work exactly in a way we end up liking. Therefore it is expected that some (if not most) of the user facing code will change and evolve over time. At the moment this means that we make no guarantees about backwards compatibility and have no formal deprecation model in place. Effectively meaning that currently we only support building from source with the latest `main` branch in our repository. We appreciate our early adopters and would ask them to evolve their code along with ours.

## Structure

There are a few different components to this repo:

- [rfcs](docs/src/rfcs) for in-depth discussion and specs.
- [rustc_codegen_spirv](rustc_codegen_spirv) for the compiler itself.
- [spirv-std](spirv-std) for GPU intrinsics, types, and other library items used by GPU crates.
- [spirv-builder](spirv-builder) for a convenient way of building a GPU crate in a CPU build.rs file.

## Contributing

[![Contributor Covenant](https://img.shields.io/badge/contributor%20covenant-v1.4-ff69b4.svg)](CODE_OF_CONDUCT.md)

We welcome community contributions to this project.

If you would like to get started, be sure to checkout the [`rust-gpu` dev guide][gpu-guide] and our [Contributor Guide](CONTRIBUTING.md) for more information on how to get started.

[gpu-guide]: https://embarkstudios.github.io/rust-gpu/

## License

Licensed under either of

- Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.

[SPIR-V]: https://en.wikipedia.org/wiki/Standard_Portable_Intermediate_Representation
