# vs-nlm-hip
Non-local means denoise filter in HIP, drop-in replacement of the KNLMeansCL for VapourSynth

## Usage
Prototype:

`core.nlm_hip.NLMeans(clip clip[, int d = 1, int a = 2, int s = 4, float h = 1.2, string channels = "AUTO", int wmode = 0, float wref = 1.0, clip rclip = None, int device_id = 0, int num_streams = 1])`

## Compilation
```bash
CMAKE_PREFIX_PATH=/opt/rocm \
CXX=/opt/rocm/llvm/bin/clang++ \
cmake -B build -S . -DCMAKE_BUILD_TYPE=Release

cmake --build build

cmake --install build
```
