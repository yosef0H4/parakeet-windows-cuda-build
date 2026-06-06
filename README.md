# parakeet.cpp Windows CUDA Build

Unofficial convenience build of [mudler/parakeet.cpp](https://github.com/mudler/parakeet.cpp) for Windows x64 CUDA.

This release is meant for people who want to run `parakeet-cli.exe` or embed `parakeet.dll` from a Rust app without spending a long time compiling ggml CUDA kernels locally.

## Release Asset

Download the release asset:

`parakeet-windows-cuda-sm86.zip`

The build targets `CMAKE_CUDA_ARCHITECTURES=86`, intended for NVIDIA Ampere laptop GPUs such as the GeForce RTX 3050 Ti Laptop GPU.

## Included

- `bin/parakeet-cli.exe`
- `bin/parakeet.dll`
- `bin/ggml*.dll`
- CUDA runtime/cuBLAS DLLs needed by the build
- Microsoft Visual C++ runtime DLLs needed by the build
- `models/tdt_ctc-110m-f16.gguf`
- `BUILD_INFO.txt`
- `THIRD_PARTY_NOTICES.txt`

## Quick Test

From the extracted folder:

```powershell
cd .\parakeet-windows-cuda\bin
.\parakeet-cli.exe info ..\models\tdt_ctc-110m-f16.gguf
.\parakeet-cli.exe transcribe --model ..\models\tdt_ctc-110m-f16.gguf --input path\to\speech.wav
```

The target machine still needs an NVIDIA driver with CUDA support. `nvcuda.dll` is not packaged because it comes from the NVIDIA driver.

## Licenses And Attribution

This is not an official release by parakeet.cpp, ggml, NVIDIA, Microsoft, or Hugging Face.

See `BUILD_INFO.txt` for the exact build details and smoke-test output.

See `THIRD_PARTY_NOTICES.txt` for model attribution, bundled runtime notes, and license references.
