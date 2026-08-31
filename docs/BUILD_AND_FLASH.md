# Odos3D-Lab — compilación y flasheo

## Objetivo fijo

Este repositorio compila para **BIGTREETECH SKR 1.4 Turbo**, cuyo MCU es **LPC1769**. El entorno por defecto de PlatformIO está fijado a `LPC1769`; `mega2560` no es un objetivo válido para esta configuración.

## Compilación local

Desde la raíz del repositorio:

```bash
python -m pip install --upgrade platformio
platformio run
```

Comando explícito equivalente:

```bash
platformio run -e LPC1769
```

Salida esperada:

```text
.pio/build/LPC1769/firmware.bin
```

La propia definición de pines de la SKR 1.4 Turbo exige LPC1769, de modo que un entorno MCU incorrecto debe detener la compilación en lugar de producir silenciosamente un firmware para otra placa.

## GitHub Actions

El workflow `.github/workflows/final-odos3d.yml` se ejecuta en cada `push` y `pull_request` hacia `main`, además de permitir ejecución manual. El flujo:

1. comprueba que `default_envs = LPC1769`;
2. verifica invariantes críticos de `Configuration.h` y `Configuration_adv.h`;
3. verifica que sensorless, BLTouch, Z2, PSU_CONTROL, StealthChop e Hybrid Threshold sigan desactivados;
4. verifica corrientes, RSENSE, microsteps, protecciones térmicas y `CHOPPER_DEFAULT_12V`;
5. compila usando el entorno por defecto con `platformio run`;
6. comprueba que exista `.pio/build/LPC1769/firmware.bin`;
7. publica un artefacto `Odos3D-Lab-firmware-LPC1769` con `firmware.bin`, `SHA256SUMS.txt` y un README que incluye el commit exacto.

## Flasheo

Usar la **microSD de la propia SKR 1.4 Turbo**, no la SD del LCD.

1. Con la máquina apagada, preparar una microSD FAT32.
2. Copiar únicamente `firmware.bin` a la raíz.
3. Insertarla en la ranura de la SKR.
4. Encender la placa.
5. Esperar el arranque completo antes de retirar alimentación.
6. Tras un flasheo correcto, el bootloader normalmente renombra el archivo a `FIRMWARE.CUR`.

El firmware queda almacenado en la memoria flash del LPC1769; la microSD no es necesaria para que el firmware siga ejecutándose después de la actualización.

## Comprobación antes de usar la máquina

Flashear correctamente no valida el cableado físico. Antes del primer homing se deben realizar las pruebas descritas en `docs/WIRING_AND_FIRST_START.md`, en especial `M122` y `M119`.

## Trazabilidad

Para una compilación final se debe conservar siempre:

- SHA del commit de `main`;
- artefacto generado por GitHub Actions;
- SHA-256 de `firmware.bin` incluido en `SHA256SUMS.txt`.

No usar binarios antiguos si existe un artefacto posterior generado desde la configuración definitiva.
