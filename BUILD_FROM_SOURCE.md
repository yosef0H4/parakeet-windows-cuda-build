# Build From Source

This document records how the Windows CUDA bundle was produced and how to rebuild it with different preferences.

The release asset is just a convenience binary. You can rebuild from upstream `parakeet.cpp` with your own CUDA architecture, model, or CUDA Toolkit version.

## Requirements

- Windows x64
- Git
- CMake
- Visual Studio Build Tools 2022 with C++ workload
- NVIDIA CUDA Toolkit
- Ninja, or the Ninja bundled with Visual Studio/CMake
- NVIDIA driver on the test machine

The published SM86 build used:

- `parakeet.cpp` commit `50dfc24b4faa4ee23a1f59401f1d0c87fc4042b0`
- CUDA Toolkit `12.8`
- MSVC `19.44.35227`
- CMake `4.3.3`
- `CMAKE_CUDA_ARCHITECTURES=86`

## Choose CUDA Architecture

Set `CMAKE_CUDA_ARCHITECTURES` for the GPU you want to support.

Common examples:

- RTX 3050 / 3050 Ti / 3060 / 3070 / 3080 Ampere: `86`
- RTX 4090 Ada: `89`
- RTX 50-series Blackwell: use the architecture supported by your CUDA Toolkit and GPU, such as `120` or toolkit-specific Blackwell values.

For a portable public build, prefer an explicit architecture instead of `native`.

## Clone

```powershell
git clone --recursive https://github.com/mudler/parakeet.cpp parakeet.cpp
cd parakeet.cpp
git checkout 50dfc24b4faa4ee23a1f59401f1d0c87fc4042b0
git submodule update --init --recursive
```

## Apply Windows Patch

The patch in this repo fixes MSVC builds where `M_PI` is not defined.

From inside `parakeet.cpp`:

```powershell
git apply ..\parakeet-windows-cuda-build\patches\0001-msvc-m-pi-fix.patch
```

If upstream has already fixed this, the patch may no longer be needed.

## Configure

Open a shell at the parent directory that contains both `parakeet.cpp` and this build-notes repo.

```powershell
cmd.exe /c "call ""<VS2022 Build Tools>\VC\Auxiliary\Build\vcvars64.bat"" && cmake -S .\parakeet.cpp -B .\parakeet.cpp\build-cuda-ninja -G Ninja -DCMAKE_MAKE_PROGRAM=""<path-to-ninja>\ninja.exe"" -DCMAKE_C_COMPILER=cl.exe -DCMAKE_CXX_COMPILER=cl.exe -DCMAKE_BUILD_TYPE=Release -DPARAKEET_BUILD_CLI=ON -DPARAKEET_SHARED=ON -DPARAKEET_GGML_CUDA=ON -DGGML_NATIVE=OFF -DCMAKE_CUDA_ARCHITECTURES=86 -DGGML_CUDA_FA=OFF -DGGML_CUDA_GRAPHS=OFF -DGGML_CUDA_NCCL=OFF -DGGML_CUDA_FORCE_CUBLAS=ON -DCMAKE_WINDOWS_EXPORT_ALL_SYMBOLS=ON"
```

Change `-DCMAKE_CUDA_ARCHITECTURES=86` if you want a different GPU target.

`CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS=ON` is used so CMake/MSVC produces `parakeet.lib` for `parakeet.dll`.

## Build

```powershell
cmd.exe /c "call ""<VS2022 Build Tools>\VC\Auxiliary\Build\vcvars64.bat"" && cmake --build .\parakeet.cpp\build-cuda-ninja --parallel 8"
```

Expected main outputs:

- `parakeet.cpp\build-cuda-ninja\examples\cli\parakeet-cli.exe`
- `parakeet.cpp\build-cuda-ninja\parakeet.dll`
- `parakeet.cpp\build-cuda-ninja\bin\ggml*.dll`

## Download Model

```powershell
New-Item -ItemType Directory -Force -Path .\models\parakeet
curl.exe -L --fail -o .\models\parakeet\tdt_ctc-110m-f16.gguf https://huggingface.co/mudler/parakeet-cpp-gguf/resolve/main/tdt_ctc-110m-f16.gguf
```

## Smoke Test

Download and convert a short speech sample:

```powershell
New-Item -ItemType Directory -Force -Path .\samples
curl.exe -L --fail -o .\samples\mlk.flac https://huggingface.co/datasets/Narsil/asr_dummy/resolve/main/mlk.flac
ffmpeg -y -i .\samples\mlk.flac -ac 1 -ar 16000 .\samples\test.wav
```

Run from the parent directory:

```powershell
$env:PATH = "$PWD\parakeet.cpp\build-cuda-ninja;$PWD\parakeet.cpp\build-cuda-ninja\bin;$env:PATH"
.\parakeet.cpp\build-cuda-ninja\examples\cli\parakeet-cli.exe info .\models\parakeet\tdt_ctc-110m-f16.gguf
.\parakeet.cpp\build-cuda-ninja\examples\cli\parakeet-cli.exe transcribe --model .\models\parakeet\tdt_ctc-110m-f16.gguf --input .\samples\test.wav
```

Expected transcript for the sample:

```text
I have a dream that one day this nation will rise up and live out the true meaning of its creed.
```

CUDA is detected if logs include lines like:

```text
ggml_cuda_init: found 1 CUDA devices
[parakeet] pk::Backend using GPU device: CUDA0
```

To force CPU for comparison:

```powershell
$env:PARAKEET_DEVICE = 'cpu'
.\parakeet.cpp\build-cuda-ninja\examples\cli\parakeet-cli.exe transcribe --model .\models\parakeet\tdt_ctc-110m-f16.gguf --input .\samples\test.wav
```

## Packaging

Create:

```text
parakeet-windows-cuda/
  bin/
  models/
  licenses/
  BUILD_INFO.txt
  THIRD_PARTY_NOTICES.txt
```

Copy into `bin/`:

- `parakeet-cli.exe`
- `parakeet.dll`
- `parakeet.lib`
- `ggml*.dll`
- `cudart64_12.dll`
- `cublas64_12.dll`
- `cublasLt64_12.dll`
- `msvcp140.dll`
- `vcruntime140.dll`
- `vcruntime140_1.dll`

Do not package `nvcuda.dll`; it comes from the target machine's NVIDIA driver.

Copy the model to:

```text
models/tdt_ctc-110m-f16.gguf
```

Include license and attribution files. The release bundle in this repo includes examples in `BUILD_INFO.txt` and `THIRD_PARTY_NOTICES.txt`.

Zip the folder:

```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::CreateFromDirectory(
  (Resolve-Path .\parakeet-windows-cuda),
  (Join-Path (Resolve-Path .) 'parakeet-windows-cuda-sm86.zip'),
  [System.IO.Compression.CompressionLevel]::Optimal,
  $true
)
```

## GitHub Release

Do not commit the zip to git. Upload it as a GitHub Release asset:

```powershell
gh release create v0.0.1-sm86 ..\parakeet-windows-cuda-sm86.zip --title "Windows CUDA SM86 build" --notes-file RELEASE_NOTES.md
```
