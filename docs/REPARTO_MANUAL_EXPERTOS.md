# Reparto manual de expertos MoE (resumen)

## El problema

Los modelos **MoE** (Qwen3-Next, DeepSeek, Mixtral...) tienen casi todo su peso
en los **expertos**: en el 80B de este proyecto, el **97%** (27,80 GB de 28,68 GB).

Cuando usas `--tensor-split` con varias GPUs, llama.cpp reparte las capas, pero
**los tensores de expertos se van todos al device 0**. Si ese device es la GPU
pequeña, pide 15 GB y revienta:

```
E ggml_backend_cuda_buffer_type_alloc_buffer:
  allocating 15121.93 MiB on device 0: cudaMalloc failed: out of memory
```

`--cpu-moe` lo evita (manda los expertos a RAM), pero entonces las GPUs solo
llevan el 3% del modelo y se desaprovechan.

## La solución (en el pack de pago)

Repartir los expertos **a mano, capa por capa**, entre las 3 GPUs con
`--override-tensor`, calculando el presupuesto de cada GPU a partir del GGUF.

Con eso, el 80B carga así:

| GPU | VRAM |
|---|---|
| RTX 5070 | 11.180 MiB |
| RTX 5060 Ti | 14.791 MiB |
| RTX 2070S (RPC) | 7.861 MiB |

**El método completo, los rangos de capas y los scripts están en el pack de pago.**
Contacto: **LordKukone** (github.com/lordkukone).
