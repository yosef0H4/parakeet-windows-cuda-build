# parakeet-windows-cuda-sm86

Unofficial Windows x64 CUDA release bundle for `parakeet.cpp`.

## Build

- Source: https://github.com/mudler/parakeet.cpp
- Commit: `50dfc24b4faa4ee23a1f59401f1d0c87fc4042b0`
- CUDA Toolkit: 12.8
- MSVC: Visual Studio Build Tools 2022, compiler 19.44.35227
- CMake: 4.3.3
- CUDA architecture: `86`
- Model: `tdt_ctc-110m-f16.gguf`

## Smoke Test

The packaged folder was tested by running `parakeet-cli.exe` from the extracted `bin` directory.

Transcription output:

```text
I have a dream that one day this nation will rise up and live out the true meaning of its creed.
```

CUDA backend was detected in the default run:

```text
ggml_cuda_init: found 1 CUDA devices
[parakeet] pk::Backend using GPU device: CUDA0
```

## Notes

- `nvcuda.dll` is not included; it is supplied by the installed NVIDIA driver.
- The package includes CUDA runtime/cuBLAS DLLs and MSVC runtime DLLs used by the build.
- See `BUILD_INFO.txt` and `THIRD_PARTY_NOTICES.txt` for full details.
