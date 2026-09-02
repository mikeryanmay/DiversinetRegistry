# DiversinetRegistry

Julia package registry for prerelease distributions of Diversinet.

## Installation

Diversinet currently supports Julia 1.12 on x86-64 and ARM64 Linux and
macOS. Add this registry once, then install Diversinet by name:

```julia
import Pkg
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

The installation downloads prebuilt native libraries through
`Diversinet_jll`; it does not require a C++ compiler, Meson, Boost, Eigen, or
a local Diversinet source checkout.
