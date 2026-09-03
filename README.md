# DiversinetRegistry

[![Clean install](https://github.com/mikeryanmay/DiversinetRegistry/actions/workflows/clean-install.yml/badge.svg?branch=main)](https://github.com/mikeryanmay/DiversinetRegistry/actions/workflows/clean-install.yml?query=branch%3Amain)

Julia package registry for prerelease distributions of Diversinet. It contains
metadata for `Diversinet` 0.1.0 and `Diversinet_jll` 0.1.0+1; source code and
binary artifacts remain in their respective public repositories.

## Installation

Diversinet currently supports Julia 1.12 on x86-64 and ARM64 Linux and
macOS. Add this registry once, then install Diversinet by name:

```julia
import Pkg
Pkg.Registry.add("General")
Pkg.Registry.add(
    Pkg.RegistrySpec(
        url="https://github.com/mikeryanmay/DiversinetRegistry.git",
    ),
)
Pkg.add("Diversinet")
```

Load the package with:

```julia
using Diversinet
```

Verify it with a small likelihood calculation:

```julia
network = "((A:0.5,B:0.5):0.25,C:0.75):0.25;"
Diversinet.computeLogLikelihood(
    network;
    λ=0.5,
    μ=0.1,
    ρ=0.5,
    kmax=16,
)
```

The expected value is approximately `-3.7565154492458603`.

The installation downloads prebuilt native libraries through
`Diversinet_jll`; it does not require a C++ compiler, Meson, Boost, Eigen, or
a local Diversinet source checkout.

The registry only provides package discovery and version resolution. It is not
a source repository, binary host, or runtime dependency. A standalone
container image is available from
[`DiversinetDocker`](https://github.com/mikeryanmay/DiversinetDocker).

## Maintenance

Register releases in dependency order: publish and register `Diversinet_jll`
first, then register the compatible `Diversinet.jl` version. Do not change the
existing package UUIDs. See `DIVERSINET_REGISTRY_MODEL.md` for the distribution
checklist and future migration plan.
