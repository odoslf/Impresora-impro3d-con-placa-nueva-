# Odos3D-Lab — auditoría de migración Impro3D

Base de destino: Marlin 2.1.2.8 estable, BTT SKR 1.4 Turbo (LPC1769), BIGTREETECH TMC2209 V1.3 UART.

Objetivo: conservar la mecánica y el comportamiento de la Impro3D original y cambiar solamente la electrónica que lo requiere. Este documento separa los valores heredados del firmware/manual original de los cambios deliberados por la nueva placa/driver o por seguridad moderna.

## Mecánica y cinemática originales — conservar

- Cartesiana, 1 extrusor, filamento 1.75 mm.
- Volumen lógico: X 0..190, Y 0..190, Z 0..190 mm.
- X/Y: GT2, poleas 20T, 80 steps/mm.
- Z: dos NEMA17 moviendo dos varillas M5, ambos motores conectados a un único driver Z. No usar Z2 independiente.
- E0 directo: 100.47095761381482 steps/mm.
- Steps/mm: `{ 80, 80, 4000, 100.47095761381482 }`.
- Max feedrate mm/s: `{ 250, 250, 3.3, 25 }`.
- Max acceleration: `{ 3000, 3000, 100, 10000 }`.
- Print acceleration: 1000 mm/s².
- Retract acceleration: 1000 mm/s².
- El firmware antiguo no tenía DEFAULT_TRAVEL_ACCELERATION independiente. En Marlin moderno se usa 1000 mm/s² como decisión de migración conservadora.
- Classic Jerk: X 20, Y 20, Z 0.4, E 5 mm/s.
- Direcciones de firmware originales: X=true, Y=false, Z=true, E0=false.
- El manual original confirma orientación física de conectores en RAMPS: cable azul abajo para X/E, arriba para Y/Z. No reutilizar esa orientación visual directamente en SKR; primero mapear bobinas/conector.

## Homing y finales de carrera originales — conservar lógica

- Home: X-, Y-, Z-.
- Homing feedrate original: `{ 2000, 2000, 150 }` mm/min.
- Bump/retract: `{ 5, 5, 1 }` mm; divisores equivalentes `{ 2, 2, 4 }`.
- ENDSTOPPULLUPS activo.
- X_MIN/Y_MIN/Z_MIN_ENDSTOP_INVERTING = true.
- Los módulos originales son finales de carrera activos de 3 hilos con LED, alimentados (VCC/GND/SIGNAL), no simples microswitches de 2 hilos.
- El manual indica cable rojo a la derecha en la RAMPS y X/Y/Z en las entradas MIN alternas.
- En SKR no se asumirá el orden físico del conector RAMPS. Se mapeará VCC/GND/SIGNAL antes de conectar.
- SENSORLESS_HOMING debe permanecer desactivado.
- Antes de cualquier G28: M119 individual, libre=open y pulsado=TRIGGERED; después verificar dirección con movimientos cortos.

## Temperatura y extrusión originales — conservar

- TEMP_SENSOR_0 = 1; TEMP_SENSOR_BED = 1.
- HEATER_0_MINTEMP = 5 °C.
- BED_MINTEMP original = 5 °C. **Desviación deliberada actual: 1 °C**, acordada para entorno frío.
- HEATER_0_MAXTEMP = 275 °C.
- BED_MAXTEMP = 95 °C.
- Hotend PID: Kp 23.05, Ki 2.00, Kd 66.47.
- Cama: bang-bang; MAX_BED_POWER 255.
- PREVENT_COLD_EXTRUSION equivalente al antiguo PREVENT_DANGEROUS_EXTRUDE.
- EXTRUDE_MINTEMP = 170 °C.
- PREVENT_LENGTHY_EXTRUDE activo.
- EXTRUDE_MAXLENGTH original efectivo = 380 mm. **Debe ser 380 en la migración; 200 del Marlin limpio no es equivalente.**
- Preheat PLA: 220/45 °C, fan 0.
- Preheat ABS: 240/70 °C, fan 0.
- EEPROM desactivada en la máquina original; mantener EEPROM_SETTINGS y EEPROM_CHITCHAT desactivados.

## Ventiladores / alimentación originales

- Tres ventiladores 40x10 estaban alimentados permanentemente con la máquina encendida.
- Blower lateral 50x15 estaba controlado por D09 en RAMPS.
- La cama tenía alimentación de potencia separada de 11A en RAMPS. La instalación actual usa MOSFET externo; Marlin sigue mandando la salida de cama de forma normal.
- El firmware antiguo contiene POWER_SUPPLY 1, pero el manual no muestra PS_ON y la UI WITBOX no ofrece control de PSU. No activar PSU_CONTROL moderno sin confirmar un cable PS_ON físico.

## Pantalla / SD / identidad

- RepRapDiscount Smart Controller 20x4 con encoder y SD.
- Idioma original: inglés.
- SDSUPPORT activo.
- Nombre original: `Prusa IMPRO3D`; splash original: `Prusa I3 IMPRO3D` + `v0.0` durante ~1.5 s.
- Nombre final acordado: **`Odos3D-Lab`**. Debe aparecer como CUSTOM_MACHINE_NAME y también en el arranque del LCD.
- La rama inicial usa por error `Odos3D.Lab`; corregir.

## Nivelado manual original WITBOX / M700

El firmware original tiene una rutina propia, no ABL:

1. Espera confirmación del usuario.
2. Home X/Y/Z.
3. Para cada punto: sube Z a 10, mueve XY, baja Z a 0.
4. Orden exacto: `(15,20)`, `(170,20)`, `(170,170)`, `(15,170)`, `(95,95)`.
5. Al terminar: Z=10 y aparca XY en `(10,10)`.
6. Los traslados XY de esta rutina usan XY_TRAVEL_SPEED = 8000 mm/min.
7. La UI original pone target de hotend a 0 y no permite iniciar el nivelado con el hotend >=60 °C; durante enfriado usa blower 255.

Marlin 2.1.2.8 `LCD_BED_TRAMMING` reproduce los cinco puntos con `BED_TRAMMING_INSET_LFRB {15,20,20,20}`, centro y Z-hop 10, pero de serie no reproduce el park final, la velocidad XY 8000 ni la protección de 60 °C. Estas diferencias deben resolverse antes de firmware final.

## Filamento original WITBOX

- Menú Load: calienta a 220 °C y después M701.
- M701 original: +100 mm de extrusión a 300 mm/min (5 mm/s), sin mover XYZ.
- Menú Unload: calienta a 220 °C y después M702.
- M702 original: +10 mm a 5 mm/s y luego -60 mm a 5 mm/s, sin mover XYZ.
- M600 original durante impresión: retract inicial 2 mm, Z +10, park X=3 Y=3, unload 100 mm, espera usuario y restaura posición/filamento al continuar.
- Marlin moderno requiere ADVANCED_PAUSE_FEATURE + NOZZLE_PARK_FEATURE. La traducción debe cuidar que M701/M702 modernos usan por defecto el Z de NOZZLE_PARK_POINT, por lo que para replicar el original se deberá usar Z0 en las acciones de carga/descarga o código equivalente.

## Drivers nuevos — valores cerrados

Hardware: 4 de los 6 BIGTREETECH TMC2209 V1.3, en X/Y/Z/E0.

- X_CURRENT = 700 mA RMS.
- Y_CURRENT = 725 mA RMS.
- Z_CURRENT = 670 mA RMS; un único driver comparte ambos motores Z.
- E0_CURRENT = 650 mA RMS.
- RSENSE = 0.11 Ω en X/Y/Z/E0.
- HOLD_MULTIPLIER = 0.5.
- Microsteps = 16; INTERPOLATE = true.
- SENSORLESS_HOMING = off.
- No BLTouch, no Z2, no Z_MULTI_ENDSTOPS, no Z_STEPPER_AUTO_ALIGN.

Estas corrientes proceden de la equivalencia ya calculada a partir de los ajustes/Vref de los A4988 originales antes de que se averiaran. No volver a inferirlas desde los 2.5 A nominales del motor.

## Diferencias modernas de seguridad aceptadas

- THERMAL_PROTECTION_HOTENDS y THERMAL_PROTECTION_BED activas.
- Watchdog moderno puede permanecer activo si compila/funciona correctamente en LPC1769.
- MONITOR_DRIVER_STATUS puede usarse para detectar condiciones de fallo TMC; revisar STOP_ON_ERROR durante validación.
- TMC_DEBUG puede habilitarse temporalmente para diagnóstico M122 y retirarse de release si se decide.

## Pendientes antes de considerar el firmware final

- [ ] Corregir nombre a `Odos3D-Lab`.
- [ ] Corregir EXTRUDE_MAXLENGTH a 380 mm.
- [ ] Replicar startup LCD `Odos3D-Lab` sin perder la base moderna de Marlin.
- [ ] Replicar exactamente Level Plate / M700 (puntos, Z-hop, velocidad XY, protección >=60 °C, park final 10,10).
- [ ] Traducir M600/M701/M702 y menús de filamento sin introducir movimientos Z que no existían en M701/M702 originales.
- [ ] Revisar PID_EXTRUSION_SCALING: el antiguo PID_ADD_EXTRUSION_RATE/Kc=1 no se copiará sin verificar equivalencia semántica.
- [ ] Decidir SpreadCycle/StealthChop para primera puesta en marcha. Prioridad: par/fiabilidad y cero pasos perdidos; el ruido es secundario.
- [ ] Confirmar jumper UART y aislamiento DIAG de TMC2209 V1.3 antes del cableado físico.
- [ ] Compilar LPC1769 tras cada lote de cambios y revisar diff.
- [ ] No fusionar PR ni flashear hasta pasar pruebas de primer encendido, M119, UART, direcciones, homing, termistores y calentadores.
