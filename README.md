# hrx-xclbinutil

A self-contained, **Boost-free**, **NPU-minimal** build of the XRT `xclbinutil`
packaging tool. It wraps a PDI (produced by bootgen) into the final `.xclbin`
container for AIE/NPU designs — with **no XRT install and no Boost**.

## Layout

```
vendor/                 trimmed XRT source (tracked), pinned to nod-ai/XRT @ a9fdf618
  xclbinutil/             30 .cxx — only the section handlers an NPU xclbin needs
                          (+ aie-pdi-transform/ = transformcdo). FPGA/flash/BMC/
                          SmartNIC/MCS sections removed.
  include/xclbin.h        axlf container format
  version.h.in            version header template
util/                   the hrx:: util module (replaces Boost)
  hrx_util.h              types + templates: hrx::format, hrx::property_tree::ptree,
                          hrx::program_options, hrx::algorithm, hrx::optional,
                          hrx::uuids, hrx::lexical_cast, ... (+ XML writer)
  hrx_util.cpp            non-template JSON read/write engine
CMakeLists.txt          self-contained build
```

The vendored source `#include "hrx_util.h"` and call `hrx::...` directly — no Boost
masquerade. Two off-path functions (`exec`, `search_path`) were reimplemented
without `boost::process`/`asio`.

## Build

Prerequisites (system): CMake ≥3.20, Ninja, and a C++17 compiler (clang or gcc) —
that's it. On Debian/Ubuntu: `sudo apt-get install cmake ninja-build clang`.
(No `uuid-dev`: the vendored `xclbin.h` self-defines `uuid_t` since only the type
is used — see the note in `vendor/include/xclbin.h`.)

```bash
cmake -G Ninja -B bld -S . -DCMAKE_BUILD_TYPE=Release
cmake --build bld --target hrx-xclbinutil
# -> bld/tools/hrx-xclbinutil   (reports "XRT Build Version: 2.18.0")
ctest --test-dir bld --output-on-failure   # functional smoke test
```

## Use

Expose it as `xclbinutil` on `PATH` (the name the AIE toolchain looks for):

```bash
mkdir -p bin && ln -sf "$PWD/bld/tools/hrx-xclbinutil" bin/xclbinutil
export PATH="$PWD/bin:$PATH"
```

## Validation

All 7 functional xclbin sections (MEM_TOPOLOGY, AIE_PARTITION, EMBEDDED_METADATA,
IP_LAYOUT, CONNECTIVITY, GROUP_TOPOLOGY, GROUP_CONNECTIVITY) are **byte-identical**
to a real-Boost build of the same tool. The only diffs are non-functional: the
axlf build timestamp (wall-clock) and a cosmetic version-metadata string (Boost's
`format("%d") % uint8` prints raw bytes; the hrx shim prints `"2.18.0"`).
