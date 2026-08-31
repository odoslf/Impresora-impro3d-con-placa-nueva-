# Odos3D-Lab — cableado y primer arranque

Este documento es la secuencia de puesta en marcha para la **SKR 1.4 Turbo + TMC2209 V1.3 UART** de Odos3D-Lab. El firmware ya está configurado para 12 V y para la mecánica existente, pero el cableado físico debe verificarse antes de aplicar potencia completa.

## Reglas antes de conectar

- Apagar completamente la fuente antes de insertar o retirar TMC2209, motores, finales, LCD, termistores o calentadores.
- No insertar ni invertir un TMC2209 con la placa alimentada.
- Confirmar la polaridad de entrada de la SKR: `+12 V` a positivo y `V- / GND` al negativo de la fuente.
- La fuente es una fuente conmutada de impresora 3D de 12 V, no una ATX; no existe cable PS_ON y `PSU_CONTROL` está desactivado.
- La tierra de protección de red/chasis no debe confundirse con el negativo de 12 V.

## Drivers TMC2209

Se usan cuatro BIGTREETECH TMC2209 V1.3:

| Zócalo | Uso | Corriente RMS |
|---|---|---:|
| X | Motor X | 700 mA |
| Y | Motor Y | 725 mA |
| Z | Dos motores Z conectados al mismo driver | 670 mA |
| E0 | Extrusor | 650 mA |

Los valores se controlan por UART desde Marlin. `RSENSE=0.11`, 16 microsteps, interpolación activa y SpreadCycle.

Antes de energizar, comprobar visualmente la orientación del driver comparando las etiquetas de alimentación y señales del módulo con las del zócalo de la SKR. No usar la orientación de los antiguos A4988 como referencia visual.

### Jumpers UART

Cada zócalo usado X/Y/Z/E0 debe quedar configurado para UART conforme al esquema de la SKR 1.4. No cambiar jumpers con alimentación aplicada.

### DIAG

`SENSORLESS_HOMING` está desactivado porque Odos3D-Lab conserva finales mecánicos. Los TMC2209 se prueban inicialmente sin cortar ninguna patilla. Antes de cualquier homing se valida el estado real de cada final con `M119`. Si DIAG interfiriese con un final, se identifica inequívocamente la patilla DIAG y se aísla solo esa señal; no se modifica STEP, DIR, EN, UART ni alimentación.

## Motores

X, Y y E0 usan un motor cada uno. Z usa **dos motores conectados al mismo driver Z**, no un driver Z2 independiente.

No asumir el orden de los cuatro hilos por color. Antes de fabricar o invertir un conector, identificar las dos parejas de bobina del motor. Cada pareja debe ocupar dos contactos correspondientes a una fase del driver. Si un eje gira al revés, primero se verifica el conector y la configuración; no se intercambia un único hilo de una bobina.

## Finales de carrera

Se mantienen los módulos mecánicos originales de 3 hilos con LED en X-, Y- y Z-.

No reutilizar a ciegas la orientación física del conector de RAMPS. Identificar en cada módulo y en la placa las funciones `SIGNAL`, `GND` y alimentación antes de conectarlo. No confiar únicamente en el color del cable.

Prueba obligatoria antes de `G28`:

```text
M119
```

Cada eje debe responder:

```text
libre   -> open
pulsado -> TRIGGERED
```

Si cualquiera queda invertido, fijo o no cambia, no hacer homing hasta resolverlo.

## Termistores y calentadores

Conectar primero los termistores y arrancar sin activar calentadores. Las temperaturas de hotend y cama deben ser plausibles y cercanas al ambiente.

Después probar por separado:

1. calentamiento suave del hotend y comprobar que sube únicamente la lectura del hotend;
2. calentamiento suave de la cama y comprobar que sube únicamente la lectura de la cama;
3. detener inmediatamente si aumenta el sensor equivocado, la lectura cae de forma anormal o aparece error térmico.

La cama conserva el MOSFET externo de potencia de la máquina. Marlin controla la salida de cama de forma normal; la etapa de potencia debe mantenerse cableada según el montaje real de la impresora.

## Ventiladores

Los tres ventiladores 40×10 originales permanecen alimentados de forma permanente cuando la máquina está encendida. El blower 50×15 de capa se conecta a la salida de ventilador controlada por firmware.

## LCD y tarjetas SD

La pantalla es RepRapDiscount Smart Controller 20×4. Verificar la orientación de EXP1 y EXP2 antes de encender con la pantalla conectada.

- microSD de la **SKR**: actualización de `firmware.bin`;
- SD de la **pantalla**: G-code durante el uso normal.

## Secuencia de primer arranque

1. Primer encendido preferente de la SKR con el mínimo hardware necesario y polaridad 12 V comprobada.
2. Flashear el `firmware.bin` generado para LPC1769.
3. Apagar.
4. Instalar y verificar drivers/jumpers/conectores.
5. Encender y ejecutar `M122`; X/Y/Z/E0 deben responder correctamente por UART.
6. Ejecutar `M119` y probar manualmente X-/Y-/Z- varias veces.
7. Hacer movimientos pequeños alejados de los topes para comprobar sentidos.
8. Solo entonces hacer homing individual y después `G28` completo.
9. El extrusor solo debe moverse con hotend por encima de `EXTRUDE_MINTEMP=170 °C`.
10. Probar hotend y cama de uno en uno antes de una impresión completa.

## Criterio de parada

Si aparece olor, humo, calentamiento anormal de placa/driver/conector, lectura térmica imposible, fallo UART, final que no cambia de estado o movimiento hacia el lado incorrecto, cortar alimentación y corregir antes de continuar.
