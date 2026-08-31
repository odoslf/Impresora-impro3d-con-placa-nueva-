# Odos3D-Lab

Firmware específico para la impresora **Odos3D-Lab / Impro3D**, migrada desde RAMPS 1.4 + A4988 a **BIGTREETECH SKR 1.4 Turbo + TMC2209 V1.3 UART**.

Este repositorio usa **Marlin 2.1.2.8** como base, pero su configuración, documentación, pruebas de CI y binario están orientados a esta máquina concreta. No es una configuración genérica de Marlin.

## Hardware objetivo

| Elemento | Configuración Odos3D-Lab |
|---|---|
| Placa | BIGTREETECH SKR 1.4 Turbo |
| MCU / PlatformIO | LPC1769 / `LPC1769` |
| Alimentación | 12 V DC |
| Drivers | 4 × BIGTREETECH TMC2209 V1.3 en UART |
| X | 700 mA RMS, 16 microsteps, SpreadCycle |
| Y | 725 mA RMS, 16 microsteps, SpreadCycle |
| Z | 670 mA RMS, un solo driver para los dos motores Z |
| E0 | 650 mA RMS, 16 microsteps, SpreadCycle |
| Motores | Wantai 42BYGHW811, 1.8° |
| Finales | X-/Y-/Z- mecánicos originales de 3 hilos con LED |
| Sensorless | Desactivado |
| BLTouch / ABL | No usado |
| Pantalla | RepRapDiscount Smart Controller 20×4 + SD |
| Cama lógica | 190 × 190 × 190 mm |
| Cama calefactada | Control Marlin normal, potencia mediante MOSFET externo de la máquina |
| Fuente | Fuente conmutada 12 V; `PSU_CONTROL` desactivado, sin PS_ON |

## Estado del firmware

La migración conserva la cinemática y los ajustes principales del firmware original: pasos/mm, direcciones, homing, finales, límites, PID del hotend, cama bang-bang y comportamiento de nivelado/filamento recuperado del firmware Impro3D.

Ajustes principales cerrados:

- `MOTHERBOARD BOARD_BTT_SKR_V1_4_TURBO`
- `default_envs = LPC1769`
- TMC2209 UART en X/Y/Z/E0
- `RSENSE = 0.11`
- `HOLD_MULTIPLIER = 0.5`
- interpolación a 256 con 16 microsteps configurados
- SpreadCycle; StealthChop e Hybrid Threshold desactivados
- `SENSORLESS_HOMING` desactivado
- `HEATER_0_MINTEMP = 1` y `BED_MINTEMP = 1`
- protecciones térmicas de hotend y cama activas
- `TMC_DEBUG` activo para disponer de `M122`
- `PSU_CONTROL` desactivado
- nombre LCD: `Odos3D-Lab`

La auditoría detallada está en [`docs/IMPRO3D_MIGRATION_AUDIT.md`](docs/IMPRO3D_MIGRATION_AUDIT.md).

## Compilar

Requisito recomendado: PlatformIO.

```bash
platformio run
```

El entorno por defecto del repositorio es **LPC1769**, por lo que el comando anterior compila directamente para la SKR 1.4 Turbo. También se puede indicar de forma explícita:

```bash
platformio run -e LPC1769
```

El binario resultante queda en:

```text
.pio/build/LPC1769/firmware.bin
```

GitHub Actions vuelve a validar los parámetros críticos y compila el mismo entorno. El artefacto de Actions es la referencia más trazable para una compilación automática de `main`.

## Flashear la SKR 1.4 Turbo

1. Formatear una microSD en FAT32.
2. Copiar `firmware.bin` a la raíz de la microSD.
3. Apagar completamente la impresora.
4. Insertar la microSD en **la ranura microSD de la propia SKR 1.4 Turbo**.
5. Encender la placa y dejar que complete el arranque.
6. Tras un flasheo correcto, el bootloader de estas placas normalmente cambia el nombre del archivo a `FIRMWARE.CUR`.

La SD de la pantalla se usa para G-code durante el funcionamiento; no debe confundirse con la microSD de la placa usada para actualizar el firmware.

## Primer encendido y puesta en marcha

No hacer `G28` inmediatamente después de montar la electrónica. La secuencia de validación del hardware está documentada en [`docs/WIRING_AND_FIRST_START.md`](docs/WIRING_AND_FIRST_START.md).

Los puntos mínimos son:

1. Alimentación apagada al insertar o retirar drivers/conectores.
2. Confirmar +12 V / V- y orientación de los TMC2209 antes de energizar.
3. Comprobar UART con `M122`.
4. Comprobar todos los finales con `M119`: libre=`open`, pulsado=`TRIGGERED`.
5. Solo después, movimientos cortos y homing individual.
6. Probar termistores antes de activar calentadores.
7. Probar hotend y cama por separado y comprobar que aumenta el sensor correcto.

### DIAG y finales originales

La máquina mantiene los finales mecánicos originales y no usa sensorless. No se debe ejecutar homing hasta verificar `M119`. La decisión física sobre aislar DIAG depende del comportamiento real de los módulos de final de carrera y se verifica durante el montaje; **no se corta ninguna patilla por defecto solo porque el firmware tenga sensorless desactivado**. Si hubiera interferencia, se aísla únicamente DIAG tras identificarlo inequívocamente.

## Documentación del proyecto

- [`docs/IMPRO3D_MIGRATION_AUDIT.md`](docs/IMPRO3D_MIGRATION_AUDIT.md): comparación y decisiones de migración.
- [`docs/BUILD_AND_FLASH.md`](docs/BUILD_AND_FLASH.md): compilación, artefactos y flasheo.
- [`docs/WIRING_AND_FIRST_START.md`](docs/WIRING_AND_FIRST_START.md): cableado y secuencia segura de puesta en marcha.
- [`release/`](release/): copia preparada para flasheo cuando se actualiza desde una compilación validada.

## Base Marlin y licencia

Este proyecto está basado en **Marlin 2.1.2.8** y conserva el código y la licencia GPLv3 de Marlin. El archivo [`LICENSE`](LICENSE) permanece como licencia del proyecto derivado. El proyecto original y su documentación están en [MarlinFirmware/Marlin](https://github.com/MarlinFirmware/Marlin) y [marlinfw.org](https://marlinfw.org/).

Las modificaciones específicas de Odos3D-Lab se mantienen en este repositorio para conservar una configuración reproducible de la máquina y cumplir con las obligaciones de distribución de la GPL.
