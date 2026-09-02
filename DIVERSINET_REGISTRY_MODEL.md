# DiversinetRegistry Distribution Model

## Goal

Distribute Diversinet through a public custom Julia registry before any package
is registered in General, while keeping the package structure ready for a
future move to General with minimal changes.

The target dependency graph is:

```text
Diversinet (C++ source repository)
        |
        v
Diversinet_jll
  - prebuilt libdiversinet
  - prebuilt libjlDiversinetInterface
        |
        v
Diversinet.jl (user-facing Julia package)

DiversinetRegistry indexes the two Julia packages above.
```

The target user installation is:

```julia
import Pkg
Pkg.Registry.add(url="https://github.com/mikeryanmay/DiversinetRegistry.git")
Pkg.add("Diversinet")
using Diversinet
```

## Phase 0: Fix Names and Permanent Identities

- [x] Use `Diversinet` as the C++ project and public repository name.
- [x] Use `Diversinet.jl` as the Julia source repository name.
- [x] Use `DiversinetJLLBuilder` for the temporary BinaryBuilder recipe and CI
      repository that stands in for Yggdrasil.
- [x] Use `Diversinet_jll` for the binary package containing both
      `libdiversinet` and `libjlDiversinetInterface`.
- [x] Use `DiversinetRegistry` for the custom registry.
- [x] Rename the local C++ directory from `diversinet_lib` to `Diversinet`, if
      desired for consistency.
- [x] Rename the GitHub C++ repository from `phyloploid` to `Diversinet`.
- [x] Update the C++ repository's `origin` remote after the GitHub rename.
- [x] Retain the existing permanent UUID for the Julia `Diversinet` package:
      `3b6113c3-62f7-4926-949c-82b48cf3552c`.
- [x] Retain the existing `Diversinet_jll` UUID if it has already been used in
      generated releases or shared environments; otherwise assign it once.
- [ ] Record all package names, UUIDs, repositories, and initial versions in
      this document before publishing releases.

### Acceptance criteria

- [ ] Every component has one unambiguous name and public repository URL.
- [x] No two packages share a UUID.
- [ ] Published UUIDs will not need to change when moving to General.

## Phase 1: Prepare the Standalone C++ Library

Repository: `Diversinet`

- [x] Confirm the Meson project is named `diversinet`.
- [x] Choose and set the first public version consistently in Meson and release
      metadata.
- [x] Retain `libdiversinet` as the installed shared-library name.
- [x] Install the public API under a conventional include path, such as:

  ```text
  include/Diversinet/DiversinetInterface.h
  ```

- [x] Ensure the installed public header is sufficient for external consumers.
- [x] Avoid requiring private `src/` headers in downstream builds.
- [x] Install `diversinet.pc` with correct include, library, and version data.
- [x] Add an OSI-approved license file to the repository
      (`GPL-3.0-or-later`).
- [x] Install the license under an appropriate distribution path
      (`share/licenses/Diversinet/LICENSE`).
- [x] Document required build dependencies:
  - [x] C++20 compiler
  - [x] Meson and Ninja
  - [x] Eigen3
  - [x] Required Boost components
- [x] Confirm the core-only build works with programs and tests disabled:

  ```sh
  meson setup build --buildtype=release -Dprograms=false -Dtests=false
  meson compile -C build
  ```

- [x] Confirm `meson install` installs only the intended public products.
- [x] Decide whether command-line programs and tests should default to enabled
      for ordinary source builds.
- [x] Add native build and test CI for every supported platform.
- [x] Remove or update stale `phyloploid` and `phyloploid_lib` names in public
      documentation and build metadata.
- [x] Create the clean source tag `v0.1.0` (currently in the private
      repository).
- [x] Publish a GitHub pre-release for `v0.1.0` (currently private).
- [x] Make the repository, tag, release, and source archive public.

### Acceptance criteria

- [x] A fresh clone builds and tests without files from another repository.
      Verified from tag `v0.1.0` with `-Dprograms=true -Dtests=true`; all four
      build targets compiled and the active analytical suite passed. The legacy
      simulation snapshot remains temporarily excluded pending replacement.
- [x] An external C++ test program can compile and link against the installed
      library.
- [x] The annotated tag `v0.1.0` is pinned to commit
      `0727a35cc367b123d7c32fe90f27e885687a8c4e` and will not be moved.
- [x] The source represented by `v0.1.0` is self-contained and builds from a
      clean checkout, making it suitable as BinaryBuilder source input.
- [x] After the repository becomes public, confirm the tagged source downloads
      without authentication. The BinaryBuilder recipe now uses the immutable
      tagged commit through `GitSource`, so no generated-archive checksum is
      required.

## Phase 2: Prepare the Julia Bridge and Build `Diversinet_jll`

The combined JLL needs bridge source from `Diversinet.jl`, while the finished
Julia package needs the combined JLL. Break that cycle in this order:

```text
Phase 2A: stabilize and tag the bridge source in Diversinet.jl
    -> Phase 2B: build and publish the combined Diversinet_jll
    -> Phase 3: make Diversinet.jl consume the published JLL
```

### Phase 2A: Stabilize the Bridge Source in `Diversinet.jl`

Repository: `Diversinet.jl`

Do not add the final `Diversinet_jll` dependency during this subphase.

- [x] Review and finalize `cpp/jlDiversinetInterface.cpp` as the canonical
      bridge source.
- [x] Keep the bridge source maintained in `Diversinet.jl`, not in the
      standalone C++ repository or generated JLL repository.
- [x] Ensure the bridge includes only the installed public C++ API
      (`Diversinet/DiversinetInterface.h`).
- [x] Ensure the bridge can be built against explicitly supplied Diversinet
      headers/library and CxxWrap headers/libraries, without relying on a
      neighboring source checkout.
- [x] Remove stale `phyloploid_lib` and obsolete local-directory references
      from bridge build metadata and public documentation.
- [x] License `Diversinet.jl` under `GPL-3.0-or-later`, matching the C++
      library.
- [x] Confirm the Julia package version is `0.1.0` and add complete `[compat]`
      bounds for the dependencies needed to build and test the bridge.
- [x] Verify the bridge still builds and initializes through the existing local
      developer workflow.
- [x] Commit the stabilized bridge source.
- [x] Record immutable Git commit
      `3c08479d4ed8286eafefc4861266d440091df9a3` as the exact bridge source
      supplied to BinaryBuilder; no intermediate Julia release tag is needed.
- [x] Create clean local bridge archive
      `sources/Diversinet.jl-3c08479d.tar.gz` from the recorded commit for
      recipe development (SHA-256
      `953359d3b9b511c8cefced6c2cf99b471a60b713f3a6268829649f98ed1108ad`);
      replace it with the public commit archive before final publication.

#### Phase 2A acceptance criteria

- [x] The bridge builds from the archived clean `Diversinet.jl` snapshot
      against an explicitly installed `libdiversinet`; the installed bridge
      also initializes successfully through CxxWrap in Julia.
- [x] The bridge source snapshot is committed and reproducible: regenerating
      it from commit `3c08479d4ed8286eafefc4861266d440091df9a3` produces the
      identical SHA-256
      `953359d3b9b511c8cefced6c2cf99b471a60b713f3a6268829649f98ed1108ad`.
- [x] No Phase 2A step requires an already-published `Diversinet_jll`.

### Phase 2B: Build and Publish the Combined `Diversinet_jll`

Recipe workspace: `DiversinetJLLBuilder`

- [x] Replace the local C++ archive with a public `GitSource` pinned to
      `Diversinet` commit `0727a35cc367b123d7c32fe90f27e885687a8c4e`
      (the commit referenced by tag `v0.1.0`).
- [x] Replace the local bridge archive with a public `GitSource` pinned to
      `Diversinet.jl` commit `3c08479d4ed8286eafefc4861266d440091df9a3`.
- [x] Verify both immutable source commits are publicly fetchable without
      authentication; `GitSource` verifies the requested Git tree directly,
      so separate archive SHA-256 hashes are not required by the recipe.
- [x] Update recipe paths to match both Git checkouts' directories.
- [x] Build and install `libdiversinet` first (verified by both Linux CI jobs).
- [x] Build `libjlDiversinetInterface` against the artifact-installed
      `libdiversinet`.
- [x] Build the bridge against the appropriate `libcxxwrap_julia_jll` headers
      and libraries.
- [x] Export both native libraries from the same JLL:

  ```julia
  products = [
      LibraryProduct("libdiversinet", :libdiversinet),
      LibraryProduct(
          "libjlDiversinetInterface",
          :libjlDiversinetInterface,
      ),
  ]
  ```

- [x] Declare all binary dependencies explicitly, including Boost and Eigen as
      appropriate.
- [x] Declare `libcxxwrap_julia_jll` as a binary dependency.
- [x] Disable command-line programs and tests in the binary recipe.
- [x] Configure portable runtime search paths so the bridge finds the adjacent
      `libdiversinet` and the artifact-provided `libcxxwrap_julia`; confirmed
      by BinaryBuilder audit on all four targets and a clean-depot load test on
      Linux x86-64 (GitHub Actions run 6).
- [x] Determine and document the supported Julia/CxxWrap compatibility range:
      CxxWrap 0.17/libcxxwrap 0.14.10 on Julia 1.12 initially.
- [x] Ensure the license is included in every artifact recipe.
- [x] Resolve the Linux `libgcc_s.so.1` audit warning by declaring
      `CompilerSupportLibraries_jll` as a runtime dependency and failing CI if
      BinaryBuilder emits the unresolved-library warning.
- [x] Remove Apple extended-attribute warnings from source archives.
- [x] Build and audit artifacts for the initial support matrix:
  - [x] macOS ARM64
  - [x] macOS x86-64
  - [x] Linux x86-64 glibc
  - [x] Linux ARM64 glibc
  - [x] Windows, or explicitly document that it is not yet supported (deferred
        for the initial release).
- [x] Confirm macOS artifacts use the intended libc++ ABI; both targets build
      with BinaryBuilder's Apple Clang toolchain and pass its artifact audit.
- [x] Confirm Linux artifacts build and pass BinaryBuilder audit with the
      `cxx11` C++ string ABI.
- [x] Deploy binaries and the generated wrapper to the public
      `Diversinet_jll` repository. The current wrapper is commit `2ac2304`
      and release `Diversinet-v0.1.0+1` (GitHub Actions run `33665143192`);
      `Diversinet-v0.1.0+0` remains the initial published build.
- [x] Assign current JLL version `0.1.0+1` (upstream `0.1.0`, second JLL
      build); increment the build suffix manually for subsequent rebuilds.
- [x] Generate the wrapper entirely through BinaryBuilder; no generated wrapper
      files are maintained or edited manually.

#### Phase 2B acceptance criteria

- [x] `using Diversinet_jll` succeeds in a clean Julia depot both for the local
      CI-generated wrapper on Linux x86-64 and for the publicly hosted package
      and artifact on macOS ARM64 with Julia 1.12.
- [x] `Diversinet_jll.libdiversinet` points to an existing artifact library on
      the native Linux x86-64 CI runner.
- [x] `Diversinet_jll.libjlDiversinetInterface` points to an existing artifact
      library on the native Linux x86-64 CI runner.
- [x] `Libdl.dlopen` succeeds for both JLL library products, and minimal
      CxxWrap initialization succeeds, on all four supported native runners
      (GitHub Actions run `33665143192`).
- [x] A minimal CxxWrap module initialization through
      `libjlDiversinetInterface` succeeds with CxxWrap 0.17.5 and Julia 1.12.6
      against the public macOS ARM64 artifact.
- [x] The bridge resolves the artifact-provided `libdiversinet`, not a locally
      installed copy; `Libdl.dllist()` reports the same canonical path as
      `Diversinet_jll.libdiversinet` under the artifact tree.
- [x] The generated JLL loads with Julia 1.12, the only Julia version currently
      claimed for this initial build.
- [x] All four artifacts pass BinaryBuilder audit without local source or
      build-machine paths (GitHub Actions run 5, recipe commit `1fe66cc`).
- [x] All four artifact download URLs are public, immutable release URLs under
      `Diversinet-v0.1.0+1`; the published artifacts passed native smoke tests
      on all four supported platforms.

## Phase 3: Convert `Diversinet.jl` to Binary Dependencies

Repository: `Diversinet.jl`

- [x] Add `Diversinet_jll` as a direct dependency.
- [x] Add `[compat]` bounds for every direct dependency.
- [x] Replace the generated `deps/deps.jl` loader with the library product
      `Diversinet_jll.libjlDiversinetInterface`.
- [x] Remove the ordinary installation dependency on `deps/build.jl`.
- [x] Remove the ordinary installation dependency on Meson and a C++ compiler.
- [x] Remove normal-use requirements for:
  - [x] `DIVERSINET_CPP_ROOT`
  - [x] `DIVERSINET_CORE_LIB`
  - [x] `DIVERSINET_CORE_INCLUDE_DIRS`
- [x] Keep native development strictly as a documented developer
      workflow, separate from package installation.
- [x] Update the README for pre-registry URL installation, future
      registry-based installation, and local Julia/native development.
- [ ] Add focused tests for:
  - [x] package loading
  - [x] network parsing
  - [x] one likelihood calculation
  - [x] one simulation
  - [ ] error handling at the Julia/C++ boundary
- [x] Add CI on every supported Julia and operating-system combination (Julia
      1.12 on Linux and macOS, x86-64 and ARM64); execution awaits review and
      push.
- [x] Confirm `cpp/jlDiversinetInterface.cpp` is unchanged from bridge source
      commit `3c08479d4ed8286eafefc4861266d440091df9a3`, which was supplied to
      the combined JLL.

### Acceptance criteria

- [x] `Pkg.add("Diversinet")` requires no compiler, Meson, Boost, Eigen, or
      local C++ checkout; verified from a clean depot against the locally
      generated registry.
- [x] `using Diversinet` succeeds from a clean depot using Julia 1.12.6.
- [x] Core functionality passes without Homebrew or system `libdiversinet`;
      both native libraries resolve inside the clean depot's artifact tree.
- [x] Moving the temporary source repository after installation does not break
      the installed package; Julia loads its immutable package copy from the
      depot and the native libraries from the artifact tree.

## Phase 4: Create `DiversinetRegistry`

Repository: `DiversinetRegistry`

- [x] Create an empty public GitHub repository named `DiversinetRegistry`.
- [x] Install `LocalRegistry.jl` in a maintainer environment, not as a
      dependency of Diversinet.
- [x] Initialize the registry with `LocalRegistry.create_registry`:

  ```julia
  using LocalRegistry

  create_registry(
      "DiversinetRegistry",
      "https://github.com/mikeryanmay/DiversinetRegistry.git";
      description="Registry for Diversinet prerelease packages",
      push=true,
  )
  ```

- [x] Verify generated registry UUID
      `bdd43285-69eb-4176-bcc1-4bf4e2c289ef` and commit it permanently.
- [x] Add a short registry README with user installation instructions.
- [x] Register packages in dependency order:
  1. [x] `Diversinet_jll` 0.1.0+1
  2. [x] `Diversinet` 0.1.0
- [x] Verify registry entries point to public Git repositories.
- [x] Verify registered versions point to committed, immutable Git trees:
      `Diversinet_jll` tree `2fb827251e972b6055ff8b376339481182e41217`
      and `Diversinet` tree `65125cc4d2ec71106c36f9332c2a10f0b618664d`.
- [x] Verify dependency UUIDs and compatibility ranges are correct.
- [x] Push all registry and installation-workflow commits to GitHub.

### Acceptance criteria

- [x] A new Julia 1.12.6 depot can add `DiversinetRegistry` from its public
      GitHub URL; add General explicitly first in a completely empty depot.
- [x] `Pkg.add("Diversinet")` resolves both packages without package URL or
      path overrides when using the locally generated registry.
- [x] General and DiversinetRegistry can be installed simultaneously; verified
      in a clean Julia 1.12.6 depot.

## Phase 5: Clean-Environment Release Testing

- [x] Test with an empty temporary Julia 1.12.6 depot.
- [x] Ensure no artifact overrides are active.
- [x] Ensure no local packages are developed into the test environment.
- [x] Ensure the system does not supply a fallback `libdiversinet`; verify the
      loaded core path is inside the clean depot's artifact tree.
- [x] Run the complete reviewer workflow:

  ```julia
  import Pkg
  Pkg.Registry.add("General")
  Pkg.Registry.add(
      Pkg.RegistrySpec(
          url="https://github.com/mikeryanmay/DiversinetRegistry.git",
      ),
  )
  Pkg.add("Diversinet")
  Pkg.test("Diversinet")
  using Diversinet
  ```

- [x] Run a representative three-taxon likelihood example (result
      `-3.7565154492458603`).
- [x] Run a rooted, tree-conditioned simulation example.
- [x] Test all supported operating systems and architectures through the clean
      public-registry workflow: Linux and macOS on x86-64 and ARM64 (GitHub
      Actions run `33673252097`).
- [x] Test the minimum supported Julia version (Julia 1.12).
- [x] Test the current supported stable Julia series (Julia 1.12).
- [x] Test a clean reinstall after removing the first depot; a second public
      registry installation and likelihood calculation passed.
- [x] Add the reviewer clean-install test to CI for Linux and macOS on x86-64
      and ARM64; execution awaits review and push.

### Acceptance criteria

- [x] A reviewer can install and run Diversinet using only Julia, Git, and
      network access.
- [x] No undocumented environment variables or system libraries are required.
- [x] CI reproduces the same public-registry installation, package tests,
      likelihood calculation, and simulation given to reviewers (GitHub
      Actions run `33673252097`).

## Phase 5A: Standalone Docker Distribution

Repository: `DiversinetDocker`

This is a downstream, optional distribution of a complete Diversinet runtime.
It installs immutable revisions of `Diversinet.jl` and `Diversinet_jll`
directly from their public GitHub repositories and does not depend on
`DiversinetRegistry`.

- [x] Create an empty local directory named `DiversinetDocker`.
- [x] Create a dedicated public GitHub repository named `DiversinetDocker` and
      connect the local `main` branch to its `origin` remote.
- [x] Add a Julia `Project.toml` and `Manifest.toml` that pin:
  - [x] `Diversinet.jl` tag `v0.1.0`;
  - [x] `Diversinet_jll` tag `Diversinet-v0.1.0+1`;
  - [x] all transitive Julia dependencies.
- [x] Add a Dockerfile based on the supported Julia 1.12 image.
- [x] Have the Docker build instantiate and precompile the committed Julia
      environment without adding `DiversinetRegistry`.
- [x] Verify `using Diversinet` during the image build.
- [x] Add a `.dockerignore` that excludes repository and development files not
      needed by the image.
- [x] Document local image build, interactive use, non-interactive use, and
      bind-mounting a host working directory.
- [x] Build and smoke-test the image locally on Linux ARM64 through Docker
      Desktop, using Julia 1.12.6 and an unprivileged container user.
- [x] Add GitHub Actions to build the image on pushes and pull requests.
- [x] Publish `latest` and commit-addressed images to GitHub Container Registry
      (`ghcr.io/mikeryanmay/diversinet`); verified by GitHub Actions run
      `33681315599` for Linux x86-64 and ARM64.
- [x] Publish versioned image `ghcr.io/mikeryanmay/diversinet:0.1.0` from the
      immutable `DiversinetDocker` tag `v0.1.0`; verified for Linux x86-64 and
      ARM64 by GitHub Actions run `33682231267`.
- [x] Document the relationship between Diversinet image versions and the
      pinned Julia/JLL revisions.
- [x] Document an image-retention and release-tagging policy in the
      `DiversinetDocker` README.

### Acceptance criteria

- [x] A user with Docker needs no local Julia, compiler, Meson, Boost, Eigen,
      custom Julia registry, or C++ checkout.
- [x] Building the same repository commit resolves the same Julia and JLL
      source revisions.
- [x] The image loads Diversinet and runs representative likelihood and
      simulation examples.
- [x] The versioned image is anonymously readable from GHCR without stored
      Docker credentials.
- [x] Docker remains an optional downstream distribution and does not alter
      normal Julia installation or future migration to General.

## Phase 6: Manuscript Reproducibility Release

- [ ] Create coordinated release versions for:
  - [ ] `Diversinet` C++ source
  - [ ] `Diversinet_jll`
  - [ ] `Diversinet.jl`
- [ ] Record the release versions and Git commit hashes in the manuscript
      repository.
- [ ] Add a reproducibility Julia environment to the manuscript repository.
- [ ] Commit its `Project.toml` and `Manifest.toml`.
- [ ] Confirm `Pkg.instantiate()` works from the committed environment.
- [ ] Archive source and release metadata with Zenodo or another suitable
      long-term archive, if required by the journal.
- [ ] Add the final installation command to the manuscript and README.
- [ ] Document the supported platforms and Julia versions.
- [ ] Document the C++ source-build procedure separately from Julia package
      installation.

### Acceptance criteria

- [ ] The manuscript identifies immutable software versions.
- [ ] Reviewers can reproduce the environment without access to local paths.
- [ ] C++ users can obtain and build Diversinet without installing Julia.

## Phase 7: Future Migration to General

No General registration is required for the initial DiversinetRegistry model.
When registration is desired later:

- [ ] Confirm all repositories use OSI-approved licenses.
- [ ] Confirm all direct dependencies have `[compat]` entries.
- [ ] Confirm package names and permanent UUIDs match DiversinetRegistry.
- [ ] Confirm clean installation and package loading under General's checks.
- [ ] Submit or migrate `Diversinet_jll` first.
- [ ] Register `Diversinet.jl` second.
- [ ] Do not change package UUIDs during migration.
- [ ] Replace custom-registry user instructions with:

  ```julia
  import Pkg
  Pkg.add("Diversinet")
  ```

- [ ] Retain DiversinetRegistry long enough for existing users to migrate.
- [ ] Decide whether to archive or continue maintaining DiversinetRegistry.

### Acceptance criteria

- [ ] Migration changes package discovery, not the dependency architecture.
- [ ] Existing tagged releases remain reproducible.
- [ ] No local path, source build, or binary-loading redesign is required.

## Release Metadata Record

Fill this table before the first public registry release.

| Component | Package UUID | Version | Repository | Source tag/commit |
|---|---|---|---|---|
| Diversinet C++ | N/A | `0.1.0` | `mikeryanmay/Diversinet` | TBD |
| DiversinetJLLBuilder | N/A | N/A | `mikeryanmay/DiversinetJLLBuilder` | TBD |
| Diversinet_jll | `c4df329e-200a-5808-ae44-368b7f2361b5` | `0.1.0+0` | `mikeryanmay/Diversinet_jll` | `bc59b3332d7a43dc303c7d42a3051f28a56c430d` / `Diversinet-v0.1.0+0` |
| Diversinet.jl | `3b6113c3-62f7-4926-949c-82b48cf3552c` | `0.1.0` | `mikeryanmay/Diversinet.jl` | TBD |
| DiversinetRegistry | TBD | N/A | `mikeryanmay/DiversinetRegistry` | TBD |

## Definition of Done

The DiversinetRegistry model is complete when a user on every supported
platform can start with a clean Julia installation and successfully run:

```julia
import Pkg
Pkg.Registry.add(url="https://github.com/mikeryanmay/DiversinetRegistry.git")
Pkg.add("Diversinet")
Pkg.test("Diversinet")
using Diversinet
```

without installing a compiler, Meson, Boost, Eigen, or a local copy of the C++
repository.
