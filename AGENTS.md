# AGENTS.md

## Cursor Cloud specific instructions

This is the **Blender** source code — a large C/C++ desktop application built with CMake. There are no web services, Docker containers, or databases; the entire product compiles into a single `blender` binary.

### Building

The canonical build entry point is the top-level `GNUmakefile`. Because the workspace root is `/workspace`, the default `BUILD_DIR` resolves to `//build_linux` (parent of `/workspace` is `/`), so **always pass `BUILD_DIR=/workspace/build`** (or equivalent).

For a cloud VM without a display, build in **headless** mode:

```
make headless ninja BUILD_DIR=/workspace/build
```

- Do **not** combine with `lite` — the lite config disables OpenEXR/OCIO/etc. but the prebuilt OpenImageIO shared library still links against them, causing undefined-reference errors at link time.
- The build uses Clang 18 (system default on Ubuntu 24.04). `libstdc++-14-dev` must be installed because Clang selects GCC 14's libstdc++ but only GCC 13's dev headers ship by default.
- Full headless build takes ~30-35 minutes on 4 vCPUs.

### Running

```
/workspace/build/bin/blender --version
/workspace/build/bin/blender --background --factory-startup --python-expr "import bpy; print(bpy.app.version_string)"
```

### Testing

```
cd /workspace/build && ctest --test-dir . -R "<pattern>" --timeout 120 --output-on-failure -j4
```

563 tests are registered. Key subsets: `script_pyapi`, `bl_rna`, `blendfile_io`, `id_management`.

### Linting

- **CMake consistency**: `make check_cmake` (pre-existing failures in HIPRT cmake files)
- **PEP8**: `make check_pep8` (runs `tests/python/pep8.py`)
- **Deprecation**: `python3 tools/check_source/check_deprecated.py`
- **Spelling**: `make check_spelling_c`, `make check_spelling_py`

See `GNUmakefile` help text (run `make help`) for the full list of targets.

### Prebuilt libraries

Prebuilt dependencies live in `lib/linux_x64/` (~2.4 GB), fetched via:

```
python3 ./build_files/utils/make_update.py --no-blender
```

Git LFS must be initialized first (`git lfs install && git lfs pull`).

### Incremental rebuilds

After code changes, re-run `make headless ninja BUILD_DIR=/workspace/build` — CMake/ninja will only recompile changed files. A clean rebuild requires `rm -rf /workspace/build`.
