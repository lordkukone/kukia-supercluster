# RESULTADOS MEDIDOS

**Fecha:** 05/11/2026 · **Por:** Xoxete · **Para:** LordKukone

---

## 1. PRUEBA DEFINITIVA: 80B EN LAS 3 GPUs ✅

**Modelo:** `Huihui-Qwen3-Next-80B-A3B-Instruct-abliterated.i1-IQ3_XXS.gguf`
**Tamaño:** 28,68 GB · **Arquitectura:** MoE (80B totales, 3B activos)

| GPU | VRAM usada |
|---|---|
| RTX 5070 (i9) | **11.188 MiB** |
| RTX 5060 Ti (i9) | **14.791 MiB** |
| RTX 2070 SUPER (i7, por RPC) | **7.843 MiB** |
| **TOTAL** | **~33,8 GB** |

### Rendimiento
```
Generación : 26,58 tok/s
Prompt     : 24,54 tok/s
Contexto   : 512
```

### Respuesta real del modelo
> "¡Hola desde las 3 GPUs! Cada una de ellas está trabajando al 100%, con los
> fans rugiendo y los núcleos brillando como estrellas..."

El modelo respondió coherente y sabiendo que estaba repartido.

---

## 2. Prueba previa: 9B Q6_K

| Dato | Valor |
|---|---|
| Modelo | Huihui-Qwen3.5-9B Q6_K (6,85 GB) |
| VRAM i7 | 7.766 MiB |
| Generación | **51,71 tok/s** |
| Prompt | 82,29 tok/s |

---

## 3. Comparativa

| Configuración | Modelo | tok/s |
|---|---|---|
| 3 GPUs (2 PCs) | 9B Q6_K | 51,71 |
| 3 GPUs (2 PCs) | 80B IQ3_XXS | 26,58 |
| CPU + `--cpu-moe` | 80B IQ3_XXS | no medido (descartado) |
| 2 GPUs del i9 (sin RPC) | 80B IQ3_XXS | **imposible** (OOM) |

El 80B **no arranca** sin la tercera GPU: necesita ~34 GB y el i9 solo tiene 28 GB.

---

## 4. Notas técnicas de la prueba

- **Contexto estable: 47.104 tokens (46K)** con `-c 47104 -ctk q8_0 -ctv q8_0 --parallel 1 -b 32 -ub 32`.
  27,7 tok/s sostenidos. 64K murió por OOM al generar
  (el buffer de cómputo del grafo de atención crece con el contexto).
- Carga desde SSD NVMe: **66-185 s** (desde HDD eran ~6 min).
- En el lanzador se pasaron **tres `--override-tensor` separados** y llama.cpp
  avisó: *"specified multiple times, use comma-separated values instead
  (only last value will be used)"*. Es decir, solo se aplicó el último (RPC0).
  Aun así, el modelo cargó y usó las 3 GPUs gracias al `--tensor-split`.
  **Mejora pendiente:** unificar los tres patrones en un solo flag separado por
  comas para aplicar el reparto exacto calculado.
- El nombre del dispositivo RPC debe ser `RPC0[192.168.50.2:50052]`.
- La comprobación válida es la VRAM de cada GPU, no `--list-devices`.

---

## 5. Hardware

| Equipo | CPU | GPU | VRAM |
|---|---|---|---|
| i9 (192.168.50.1) | Intel i9-10900K | RTX 5070 + RTX 5060 Ti | 12 + 16 GB |
| i7 (192.168.50.2) | Intel i7 | RTX 2070 SUPER | 8 GB |
| Red | 2,5 Gbps | — | 283 MB/s medidos |

---

*Mediciones reales, no estimaciones.*
