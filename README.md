# Efficient Real-time Quadcopter Propeller Detection and Attribute Estimation

This repository contains the official C++ implementation of the paper **"Efficient Real-time Quadcopter Propeller Detection and Attribute Estimation with High-resolution Event Camera"** accepted to Scandinavian Conference on Image Analysis, to appear in proceedings of the conference. The code detects rotating propellers in high-resolution event camera streams and estimates their angular speed as well as the relative pitch and roll of the propeller plane.

## Overview

The detector processes asynchronous events from a Metavision-compatible camera. Events are binned into blocks of size `2^k` and modeled as Poisson processes. Real-time bursts in the inter-event statistics reveal rotating propellers. Once a propeller is detected, its angular speed is computed from inter-burst intervals, and an ellipse fit over accumulated events yields pitch and roll estimates.


## Repository Layout

- `main.cpp` – application entry point that loads a YAML configuration and runs the detector.
- `QuadcopterPropellerDetector_lib/` – library implementing the Poisson based burst detection and ellipse fitting algorithms.
- `config.yaml` / `config_win_detector.yaml` – example configuration files used to specify input recordings, output paths and processing parameters.
- `compile_build.sh` – script used on our environment for building inside an Apptainer container.
- `Google_tests/` – minimal Google Test setup.

## Building

The project uses CMake (≥3.26) and requires a C++17 compiler. Dependencies include
[Metavision SDK](https://www.prophesee.ai/metavision/), Boost, yaml-cpp, HDF5 and
Protobuf. 

Before any attempt to build, one has to correct all absolute paths in all `CMakeLists.txt` files.

Linking local HDF5 libraries and MetavisionSDK is a pain. I am willing to help any poor soul wishing to do so.

On Linux the following commands build the executable `qpdet`:

```bash
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j
```

On Windows with Visual Studio generator, build using:

```bash
cmake --build . --config Release
```

A helper script `compile_build.sh` is also provided for building inside an Apptainer/Singularity container.

## Running

The detector is invoked with a YAML configuration file:

```bash
./qpdet config.yaml
```

The configuration specifies the path to a RAW event recording (`recording_filepath`),
and controls window size, burst-detection thresholds, and whether results are visualized or stored for analysis.

## 配置清单（需要自行修改的路径/参数）

### 1) 构建依赖路径（CMake）

Windows 下这些路径是硬编码的，必须按你的安装位置修改：

- [CMakeLists.txt](CMakeLists.txt)
  - Prophesee SDK 根路径：`c:/Program Files/Prophesee/...`
  - Boost：`BOOST_ROOT`、`Boost_INCLUDE_DIR`、`Boost_LIBRARY_DIR`
  - TIFF：`TIFF_INCLUDE_DIR`、`TIFF_LIBRARY`
  - ZLIB：`ZLIB_INCLUDE_DIR`、`ZLIB_LIBRARY`
  - Protobuf：`Protobuf_INCLUDE_DIR`、`Protobuf_LIBRARIES`
  - HDF5 头文件：`HDF5_INCLUDE_DIRS`
- [QuadcopterPropellerDetector_lib/CMakeLists.txt](QuadcopterPropellerDetector_lib/CMakeLists.txt)
  - Boost：`BOOST_ROOT`、`Boost_INCLUDE_DIR`、`Boost_LIBRARY_DIR`
  - TIFF：`TIFF_INCLUDE_DIR`、`TIFF_LIBRARY`
  - HDF5 头文件：`HDF5_INCLUDE_DIRS`

Linux 下默认使用系统路径（如 `/usr/include/hdf5/serial`、`/usr/include/metavision`、`/usr/lib/metavision`），如果你的安装位置不同，同样需要在 [CMakeLists.txt](CMakeLists.txt) 和 [QuadcopterPropellerDetector_lib/CMakeLists.txt](QuadcopterPropellerDetector_lib/CMakeLists.txt) 中调整。

> 提示：若你使用 HDF5 的 CMake 配置包，可以在配置时显式传入 `-DHDF5_DIR=<hdf5-config.cmake 所在目录>` 来避免找错版本。

### 2) 运行配置（YAML）

必须修改为你本机可用的路径：

- [config.yaml](config.yaml)
  - `recording_filepath`：RAW 事件流数据路径
  - `analysis_filepath`：分析输出 .h5 文件路径
  - `save_bbox`：检测框输出 .csv 文件路径
  - `tensorboard_log_file`：TensorBoard 日志文件名（相对路径/工作目录）
- [config_win_detector.yaml](config_win_detector.yaml)
  - `recording_filepath`：RAW 事件流数据路径（Windows 示例）
  - `analysis_filepath`：分析输出 .h5 文件路径
  - `tensorboard_log_file`：TensorBoard 日志文件名（相对路径/工作目录）

确保输出目录已存在（如 `results/`），否则运行时写文件会失败。

### 3) 数据集路径

示例配置中假定数据集位于本机磁盘路径，需替换为你的实际下载位置。数据集下载地址见上文 Dataset。

### 4) 容器构建脚本（可选）

如果使用 [compile_build.sh](compile_build.sh)，需要修改以下路径：

- `--bind /mnt/data/vrg/spetlrad/data:/mnt/data/vrg/spetlrad/data`
- 容器镜像：`/mnt/data/vrg/spetlrad/data/singularity/ptbd_pop.sif`

### 5) 清理旧缓存

在不同系统/路径间切换构建时，需要删除 `build/` 或至少移除 `build/CMakeCache.txt` 和 `build/CMakeFiles/`，避免缓存里的旧路径导致配置失败。

## Dataset

The repository is accompanied by a dataset of quadcopter propeller recordings covering rotational speeds from 1100–8200 RPM and tilt angles of 0°, 10°, and 90°.

The dataset is stored at the [propeller dataset download](https://ptak.felk.cvut.cz/personal/spetlrad/propeller_dataset.tar.gz).

Our detector achieves near-perfect detection accuracy with an average real‑time factor of 0.94 on a single CPU core.

## Paper

```bibtex
@inproceedings{author2024efficient,
  title     = {Efficient Real-time Quadcopter Propeller Detection and Attribute Estimation with High-resolution Event Camera},
  author    = {Spetlik, Radim and Uhrova, Tereza and Matas, Jiri},
  booktitle = "Image Analysis",
  year      = {2025}
}
```

Please cite the paper if you use this code or dataset.

