# Collaborator build and run guide

This repository contains the effqpdet source. RAW recordings, annotations, SDK binaries, and generated analysis files are external inputs and are not stored here.

## Requirements

- A C++17 compiler and CMake 3.26 or newer.
- A compatible Metavision SDK, Boost `program_options`, `yaml-cpp`, HDF5, and Protobuf installation.
- The exact Metavision and HDF5 versions used for the local Windows build were not recoverable, so this repository does not provide a complete environment lock.

The upstream CMake files may contain machine-specific include or package paths. Review them and configure the paths for the installed compiler and SDK before building.

## Configuration

Copy `source/config_collaborator_template.yaml` or edit it for the intended recording. Set `recording_filepath` to a RAW file you are authorized to use. The output paths are relative to the working directory and point under `output/`.

The historical `source/config.yaml` and files in `source/yaml/` are provenance snapshots from other machines. Review every input path before using them. No RAW file is bundled with this repository.

## Build and run

From the repository's `source` directory:

```powershell
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
.\build\Release\qpdet.exe .\config_collaborator_template.yaml
```

For a short local smoke run, set an appropriate short `end_us` in the configuration. That does not replace the full experiment duration. Analysis output is enabled by the template and can create large HDF5 files under `source/output/`; these outputs are generated artifacts.

## Experiment provenance

The local working tree's `Detector.cpp` and `types.h` were checked against their Git blobs and contained no source-code delta. The two local run configurations with absolute recording paths were excluded from this repository. The template above removes those paths and uses placeholders.
