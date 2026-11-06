# Scripts — versión de pago

Los scripts que hacen funcionar el reparto de expertos MoE entre las 3 GPUs
**no están en este repositorio público**.

## Qué incluye el pack de pago

| Script | Qué hace |
|---|---|
| `reparto_manual.py` | Calcula los rangos de capas y genera los flags `--override-tensor` |
| `medir_expertos.py` | Mide cuánto pesan los expertos y la atención de un GGUF |
| `analizar_capas.py` | GB por capa del modelo |
| `lanzar_80b_manual.bat` | Lanzador probado del 80B con las 3 GPUs |
| `lanzar_9b_3gpu.bat` | Ejemplo con un 9B (51,71 tok/s) |
| `instalar_en_i7.ps1` | Copia binarios + DLLs al PC remoto y arranca el RPC |
| `probar_rpc.ps1` | Copia, prueba y lanza |
| `configurar.bat` / `compilar.bat` | Compilación automática |
| `arrancar_rpc.bat` | Arranque del RPC en el PC remoto |

## Precio

**149 €** · Contacto: **kukone510@hotmail.com** — github.com/lordkukone

La compra incluye:
- Los scripts completos
- La guía maestra de 22 KB (21 apartados, paso a paso)
- La receta del reparto manual de expertos
- Soporte por correo

---

Mientras tanto, puedes compilar llama.cpp con RPC siguiendo
[`../docs/COMPILAR.md`](../docs/COMPILAR.md) — eso sí es gratis.
