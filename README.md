# Cómo instalar LineageOS 20 (GSI) con TWRP y Heimdall en el Samsung Galaxy A04e (SM-A042M)

Guía paso a paso, en español, de una instalación **hecha y probada en un equipo real**. Úsala bajo tu propio riesgo: flashear puede dejar el celular sin arrancar y borra todos los datos.

> 🇬🇧 English summary at the [bottom](#english-summary).

## Resumen rápido
1. Respaldar particiones críticas y datos (¡con IMEI incluido!).
2. Flashear **TWRP** con **Heimdall** desde modo descarga.
3. Entrar a TWRP, **formatear data**.
4. Grabar el GSI **`arm64_bvN`** de LineageOS 20 en `system`.
5. Reiniciar. Arranca en unos minutos.

**Resultado:** cámara, audio, wifi, Bluetooth y táctil funcionan. Red móvil sin probar.

## Índice
[Equipo](#equipo-donde-se-probó) · [La clave del nombre "A-only"](#la-clave-que-casi-nos-confunde) · [Archivos](#archivos-usados) · [Antes de empezar](#antes-de-empezar) · [Paso a paso](#pasos) · [Qué funciona](#qué-se-comprobó) · [Rendimiento](#rendimiento) · [Táctil](#pantalla-táctil-lenta-tras-cambiar-la-pantalla) · [Advertencias](#advertencias)

## Equipo donde se probó
| Dato | Valor |
|---|---|
| Modelo | SM-A042M (código `a04e`), chip MediaTek Helio P35, 2,8 GB de RAM |
| Firmware de fábrica | `A042MUBSDEYG1`, Android 14, parche 2025-07-01 |
| Treble | activado, VNDK 31, particiones dinámicas (`super`), system-as-root |
| Bootloader | desbloqueado de antemano, root con Magisk |
| Resultado | **LineageOS 20 (Android 13) arrancó** |

## La clave que casi nos confunde
Treble Info dice "A-only" porque el celular **no tiene doble slot (A/B)**. Pero como usa particiones dinámicas y system-as-root, le corresponde un GSI de tipo **`b`** (por ejemplo `arm64_bvN`), no uno `a`. Las imágenes `a` son solo para celulares antiguos sin system-as-root. Los nombres A-only de AndyYan no existen para Android 12+ y no hacen falta.

## Archivos usados
| Archivo | Fuente | Verificación |
|---|---|---|
| `twrp-3.7.1_12-0-a04e.img.tar` (~37 MB) | [dl.twrp.me/a04e](https://dl.twrp.me/a04e/) | SHA-256 `97d74e36f89d2c24575ccb7d816242df5b07f63691604c4804b4ece08c95c1ce` |
| `lineage-20.0-20251021-UNOFFICIAL-arm64_bvN.img.gz` (~996 MB) | [AndyYan en SourceForge](https://sourceforge.net/projects/andyyan-gsi/files/lineage-20-td/) | descomprimido (2 242 465 792 bytes): SHA-256 `ed05812760984af2a36514ceb487fde58705de636a2f849a6722e59d5016baf4` |
| `heimdall` 2.2.2 | repos de tu distro | `heimdall version` |

> Imagen **vanilla** (sin GApps) y sin superusuario integrado (`N`). El root vino del Magisk que ya estaba en `boot`.

## Antes de empezar
1. Batería cargada (lo recomendado es más del 80 %).
2. Depuración USB activada y autorizada: `adb devices` debe decir `device`.
3. **Respalda estas particiones con root**: `boot recovery vbmeta vbmeta_system vbmeta_vendor dtbo lk efs sec_efs nvram nvdata nvcfg persist persistent proinfo protect1 protect2 param up_param seccfg`. Con `dd if=/dev/block/by-name/<p> of=...` y `adb pull`.
   **No subas nunca esas copias a internet**: contienen tu IMEI y calibración, y no se pueden recuperar si se pierden.
4. Copia tus archivos personales (`adb pull /sdcard/.`). Las particiones de `super` no se respaldaron, así que ten a mano el firmware de fábrica de tu modelo para volver atrás con Odin.

## Pasos
**1. Modo descarga**
`adb reboot download` (o apagado, Vol+ y Vol− y conectar el cable).

**2. Tabla de particiones** (opcional, para comprobar nombres)
`heimdall download-pit --output a04e.pit --no-reboot`
La partición se llama `recovery`, en **minúscula**.

**3. Flashear TWRP**
`tar -xf twrp-3.7.1_12-0-a04e.img.tar` (sale `recovery.img`)
`heimdall flash --recovery recovery.img --no-reboot`

> ⚠️ **Heimdall solo funciona en la primera sesión después de reconectar el cable.** Si falla con `Protocol initialisation failed`, desconecta y vuelve a conectar el USB sin salir de modo descarga y que el flasheo sea lo primero que ejecutes.
> `--no-reboot` es importante: si el celular arranca Android, Samsung reemplaza TWRP por su recovery.

**4. Entrar a TWRP de inmediato**
Mantén Vol− + Encendido hasta que se apague la pantalla. En cuanto se apague, suelta y mantén de inmediato **Vol+ + Encendido** hasta ver TWRP. Elige *Swipe to Allow Modifications*.
Con el USB conectado, `adb devices` debe mostrar el equipo en modo `recovery`.

**5. Borrar datos**
En TWRP: *Wipe → Format Data → `yes`*. (Se hizo desde la pantalla; el formateo remoto por `adb` fue bloqueado por el entorno.)

**6. Subir y flashear el GSI**
```
gzip -dc lineage-20.0-...-arm64_bvN.img.gz > system.img
adb push system.img /data/system-gsi.img
adb shell dd if=/data/system-gsi.img of=/dev/block/mapper/system bs=4M
adb shell sync
```
La partición `system` del A04e mide 3,92 GB y la imagen 2,24 GB, así que cabe. Se verificó con `sha256sum` que lo grabado coincide con la imagen.

**7. Reiniciar**
`adb reboot`. El primer arranque tarda unos minutos.

## Qué se comprobó
| Función | Resultado |
|---|---|
| Arranque de LineageOS 20 | ✅ funciona, sin crashes en el log de arranque |
| Wifi | ✅ adaptador activo y salida a internet |
| Bluetooth, sensores (7) | ✅ |
| Cámara | ✅ probada por el usuario (el sistema detecta 2 cámaras) |
| Audio | ✅ probado por el usuario |
| Pantalla táctil | ✅ responde bien con la alta sensibilidad activada (confirmado por el usuario) |
| **Red móvil / SIM / datos** | ❓ **sin probar** (el equipo no tenía chip). En el hermano A04 con GSI de Lineage 21 se reportó que **no hay datos móviles** ([phhusson#2623](https://github.com/phhusson/treble_experimentations/issues/2623)) |

## Rendimiento
Con Lineage hay ~1,4 GB de RAM libres, frente a ~850 MB con el One UI de fábrica. Ajustes aplicados: animaciones a 0,5x y desactivar el asistente de configuración (`pm disable-user --user 0 org.lineageos.setupwizard`).

## Pantalla táctil lenta tras cambiar la pantalla
Si el panel es `TXD_BOE_JD9365T` (Jadard), con root:
```
echo 1 > /proc/jadard_touch/SENSITIVITY
```
Para que sobreviva al reinicio, un script de Magisk en `/data/adb/service.d/touch-sens.sh`:
```
#!/system/bin/sh
sleep 20
echo 1 > /proc/jadard_touch/SENSITIVITY
```
El modo viene desactivado (`High_sensitivity_enable=0`) en este panel. Al activarlo, el usuario confirmó que el táctil responde bien (es una apreciación, no una medición). Si no ayuda en tu caso, el problema puede ser la pantalla de repuesto o su conector.

## Advertencias
- Solo se probó con **este modelo y este firmware**. Otros SM-A042x pueden diferir.
- TWRP 3.7.1_12 para a04e no está bien soportado en todas las versiones; XDA lo reporta como poco fiable. Aquí funcionó.
- No se tocó `vbmeta`: no hizo falta, el bootloader ya estaba desbloqueado (estado *orange*).
- Los datos móviles pueden no funcionar. Prueba con una SIM antes de depender del celular como principal.
- Esto borra todos los datos y puede anular la garantía.

## English summary
Installing a **LineageOS 20 GSI** (Android 13) on the **Samsung Galaxy A04e (SM-A042M, `a04e`, MediaTek Helio P35)** with **TWRP 3.7.1** and **Heimdall**. Worked on firmware `A042MUBSDEYG1` (bootloader already unlocked).
- Treble Info says "A-only" because there is no A/B slot, but the device uses dynamic partitions + system-as-root, so the correct image is a **`b`** GSI (`arm64_bvN`), not an `a` one.
- Heimdall only works on the **first session after replugging the cable**; run the flash command first. Partition name is lowercase `recovery`.
- Steps: backup (never publish IMEI/`nvram`/`persist`) → flash TWRP (`heimdall flash --recovery recovery.img --no-reboot`) → boot TWRP → Format Data → `dd` the decompressed GSI to `/dev/block/mapper/system` → reboot.
- Result: boots; camera, audio, wifi, Bluetooth, touch OK. **Mobile network/SIM untested.** A similar A04 reported no mobile data on LineageOS 21 GSI.
- Touch lag after a screen replacement (Jadard panel): `echo 1 > /proc/jadard_touch/SENSITIVITY`.
Full commands are in the Spanish steps above.

## Créditos y fuentes
- [TWRP para a04e](https://twrp.me/samsung/samsunga04e.html) · [GSI de AndyYan (TrebleDroid)](https://sourceforge.net/projects/andyyan-gsi/) · [Heimdall](https://github.com/Benjamin-Dobell/Heimdall)
- Hilo de XDA que inspiró esto: [How to install TWRP and GSIs in the A04e](https://xdaforums.com/t/custom-recovery-gsi-how-to-install-twrp-and-gsis-in-the-a04e.4640124/)
