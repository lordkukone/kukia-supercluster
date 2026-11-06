# Compilar llama.cpp con RPC en Windows (guía gratuita)

Esta es la parte gratuita: **compilar con el backend RPC activado**.
El reparto manual de expertos MoE (la parte difícil) está en el pack de pago.

---

## 1. Requisitos

| Programa | Versión probada |
|---|---|
| Git | 2.55.0 |
| CMake | 4.4.4 |
| Visual Studio Build Tools 2022 | 17.14.44 (MSVC 19.44) |
| CUDA Toolkit | 13.4 |
| Ninja | incluido en VS |

Instalación:

```powershell
winget install --id Kitware.CMake -e --accept-source-agreements --accept-package-agreements --silent
winget install --id Microsoft.VisualStudio.2022.BuildTools -e `
  --accept-source-agreements --accept-package-agreements `
  --override "--quiet --wait --norestart --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"
```

---

## 2. Clonar

```powershell
git clone --depth 1 https://github.com/ggml-org/llama.cpp.git C:\LLAMA_RPC\llama.cpp
```

---

## 3. Configurar

Crear `C:\LLAMA_RPC\configurar.bat`:

```bat
@echo off
call "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\Common7\Tools\VsDevCmd.bat" -arch=x64 -host_arch=x64
set PATH=C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\Ninja;%PATH%

cmake -S C:\LLAMA_RPC\llama.cpp -B C:\LLAMA_RPC\build_ninja -G Ninja ^
  -DCMAKE_BUILD_TYPE=Release ^
  -DGGML_CUDA=ON ^
  -DGGML_RPC=ON ^
  -DCMAKE_CUDA_ARCHITECTURES=75;120 ^
  -DCMAKE_CUDA_COMPILER="C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v13.4/bin/nvcc.exe" ^
  -DLLAMA_CURL=OFF -DLLAMA_BUILD_TESTS=OFF -DLLAMA_BUILD_EXAMPLES=OFF
```

```powershell
cmd /c "C:\LLAMA_RPC\configurar.bat > C:\LLAMA_RPC\configurar.log 2>&1"
```

**Importante:** `75` = Turing (RTX 2070S) · `120` = Blackwell (RTX 5070/5060 Ti).
Ajusta a tus GPUs.

---

## 4. Compilar

```bat
cmake --build C:\LLAMA_RPC\build_ninja -j 12
```

444 objetos · ~20-25 min.

---

## 5. Preparar el PC remoto

```powershell
Copy-Item "C:\LLAMA_RPC\build_ninja\bin\*" "\\192.168.50.2\F$\llama-rpc-new" -Recurse -Force
Copy-Item "C:\LLAMA\cublas64_13.dll","C:\LLAMA\cublasLt64_13.dll","C:\LLAMA\cudart64_13.dll","C:\LLAMA\libomp.dll" "\\192.168.50.2\F$\llama-rpc-new" -Force
```
Sin esas 4 DLLs el servidor RPC **no arranca** (log vacío).

```powershell
# En el i7:
cd /d F:\llama-rpc-new
ggml-rpc-server.exe -H 0.0.0.0 -p 50052
```

Debe mostrar:
```
Starting RPC server v8.0.1
Devices:
  CUDA0: NVIDIA GeForce RTX 2070 SUPER (8191 MiB)
```

---

## 6. Lanzar (sin reparto manual)

```powershell
C:\LLAMA_RPC\build_ninja\bin\llama-server.exe -m "RUTA\MODELO.gguf" `
  -ngl 999 --rpc 192.168.50.2:50052 --tensor-split 8.4,16,6.8 `
  -c 512 --port 8125 --host 127.0.0.1
```

- Con modelos **densos** (no MoE) esto puede bastar.
- Con modelos **MoE** (Qwen3-Next, DeepSeek, Mixtral...) los expertos se irán
  todos al device 0 y fallará. Ese es el problema que resuelve el pack de pago.

---

## 7. Comprobar

**No uses `--list-devices`** (no conecta a los RPC). Mira la VRAM:

```powershell
nvidia-smi --query-gpu=index,memory.used --format=csv,noheader
Invoke-Command -ComputerName 192.168.50.2 -Credential $cred -ScriptBlock { nvidia-smi }
```

Si la VRAM del i7 sube, el RPC funciona.
