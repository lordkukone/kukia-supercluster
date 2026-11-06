# CONFIGURACION DEL 80B EN 3 GPUs — KUKIA SUPERCLUSTER

**Probado:** 06/11/2026
**Resultado:** 61.440 tokens de contexto · **59,1 tok/s**

---

## El comando exacto

```powershell
C:\LLAMA_RPC\build_ninja\bin\llama-server.exe `
  -m "C:\MODELOS\Huihui-Qwen3-Next-80B-A3B-Instruct-abliterated.i1-IQ3_XXS.gguf" `
  -ngl 999 `
  --rpc 192.168.50.2:50052 `
  --tensor-split 21,31,48 `
  -c 61440 `
  --cache-type-k q4_0 --cache-type-v q4_0 `
  --batch-size 128 --ubatch-size 64 `
  --repeat-penalty 1.1 `
  --repeat-last-n 256 `
  --dry-multiplier 0.8 `
  --dry-base 1.75 `
  --dry-allowed-length 2 `
  --dry-penalty-last-n 2048 `
  --port 8250 `
  --host 127.0.0.1 `
  --no-warmup
```

**O simplemente:** `C:\KUKIA_INFORME\LANZAR_80B.bat`

---

## Los parámetros, uno a uno

| Parámetro | Valor | Por qué |
|---|---|---|
| `-ngl 999` | todas las capas | Todo en GPU, ninguna en CPU |
| `--rpc` | `192.168.50.2:50052` | La 2070 SUPER del i7 |
| `--tensor-split` | `21,31,48` | **Orden: RPC0, CUDA0, CUDA1** |
| `-c` | `61440` | 60K de contexto |
| `--cache-type-k/v` | `q4_0` | KV a 4 bits: más contexto Y más rápido |
| `--batch-size` | `128` | Lote de prompt |
| `--ubatch-size` | `64` | Sublote |
| `--repeat-penalty` | `1.1` | Evita repetir tokens |
| `--repeat-last-n` | `256` | Mira 256 atrás |
| `--dry-multiplier` | `0.8` | **Evita repetir frases enteras** |
| `--dry-penalty-last-n` | `2048` | Mira 2048 atrás para DRY |
| `--host` | `127.0.0.1` | Solo local (cambiar a `0.0.0.0` para acceso desde otros) |

---

## Reparto de VRAM

| Device | GPU | Split | VRAM real |
|---|---|---:|---:|
| **RPC0** | RTX 2070 SUPER (i7) | 21 % | 7.831 MB |
| **CUDA0** | RTX 5070 (i9) | 31 % | 11.272 MB |
| **CUDA1** | RTX 5060 Ti (i9) | 48 % | 14.142 MB |
| | **TOTAL** | | **33.245 MB** (90 %) |

---

## Los dos perfiles

| Perfil | Contexto | KV | Velocidad | Comando |
|---|---:|---|---:|---|
| **Calidad** | 47.104 | q8_0 | 46,1 tok/s | `-c 47104 --cache-type-k q8_0 --cache-type-v q8_0` |
| **Cantidad** | **61.440** | **q4_0** | **59,1 tok/s** | el de arriba |

**El q4_0 gana en las dos cosas**: más contexto Y más rápido.

---

## Cómo saber qué contexto cabe

```
VRAM libre total - 28.680 (modelo) - 1.000 (buffers) = MB para KV

MB por token:
  f16   : 0,167
  q8_0  : 0,0835
  q4_0  : 0,0418   ← el que usas
  q3_K  : 0,0313
  q2_K  : 0,0209

Contexto = MB_para_KV / MB_por_token
```

Con 4.050 MB para KV:
- q8_0 → 48.503 tokens
- **q4_0 → 96.890 tokens** (limitado a 61.440 por VRAM real disponible)

---

## Antes de lanzar: cerrar esto

Hacen falta ~30 GB libres de 36,7.

| Cerrar | Libera |
|---|---:|
| Steam, Epic, Battle.net, EA | ~1 GB |
| Discord, WhatsApp, BlueStacks | ~0,5 GB |
| Edge (todos los procesos) | ~3 GB |
| TTS3 (qwen3-tts-studio) | ~4 GB |
| ProtonVPN, NZXT CAM, Signal | ~0,5 GB |
| **llama-server huérfano del i7** | **5,8 GB** |

---

## Errores conocidos

| Error | Causa | Solución |
|---|---|---|
| `couldn't bind HTTP server socket` | Puerto **reservado** por Windows | Usar 8250. Ver `netsh int ipv4 show excludedportrange protocol=tcp` |
| `cudaMalloc failed: out of memory` | Reparto mal calculado | El split debe ser proporcional a la VRAM **libre** |
| `Failed to connect to 192.168.50.2:50052` | rpc-server caído | `vigilar_rpc.ps1` lo levanta |
| Modelo repite frases | Sin penalización | `--repeat-penalty 1.1 --dry-multiplier 0.8` |
| `invalid dry-penalty-last-n = -1` | Valor no válido | Usar `2048` (o 0 para desactivar) |
| Solo veo 2 GPUs en `--list-devices` | Ese comando no conecta al RPC | Lanzar con `-v` y buscar `RPC0` |

---

## Comprobaciones antes de cargar

```powershell
# RPC del i7
Invoke-Command -ComputerName 192.168.50.2 -ScriptBlock {
  Get-NetTCPConnection -LocalPort 50052 -State Listen
}
Test-NetConnection -ComputerName 192.168.50.2 -Port 50052

# VRAM de las 3
nvidia-smi --query-gpu=index,memory.used,memory.free --format=csv,noheader
Invoke-Command -ComputerName 192.168.50.2 -ScriptBlock {
  & "C:\Windows\System32\nvidia-smi.exe" --query-gpu=memory.used,memory.free --format=csv,noheader
}

# ¿Huérfanos?
Get-Process llama-server -ErrorAction SilentlyContinue
Invoke-Command -ComputerName 192.168.50.2 -ScriptBlock {
  Get-Process llama-server -ErrorAction SilentlyContinue
}
```

---

## Acceso

```
http://127.0.0.1:8250      (desde el i9)
```

**Aviso sobre el host web**: el campo **"Max Tokens"** de la interfaz corta la
generación en 5.000 por defecto. Cambiarlo a `60000` o `-1`.

Por API (`/v1/chat/completions`) no hay ese límite: se pone `max_tokens` en el body.

---

*Documentado por Xoxete el 06/11/2026*
