# LEVELS — PCB planta baja (v0.1)

> **Nota:** los archivos de KiCad y los Gerbers que se listan abajo todavía no están en este repositorio. Cuando se agreguen, irán en la carpeta `hardware/`.

Placa de 70 × 45 mm, 2 capas. Una sola placa para las dos versiones:

- **1 sensor (tinaco):** soldar todo excepto J2, R3, R4, R6, C3, C4 → firmware [`tinaco.yaml`](../firmware/tinaco.yaml)
- **2 sensores (tinaco + cisterna):** soldar todo → firmware [`tinaco_cisterna.yaml`](../firmware/tinaco_cisterna.yaml)

Formato KiCad 7, así que abre en KiCad 7, 8 y 9 (al guardar se convierte a tu versión).

## Contenido

| Archivo | Qué es |
|---|---|
| `LEVELS.kicad_pro` | Proyecto: ábrelo primero |
| `LEVELS.kicad_sch` | Esquemático (conexiones con etiquetas de red) |
| `LEVELS.kicad_pcb` | PCB ruteada, con planos de GND y barrenos M3 |
| `gerbers/` | Gerbers y taladros listos para fabricar |
| `render_cara_superior.png`, `render_cara_inferior_espejo.png`, `LEVELS_pcb_capas.pdf` | Vistas de revisión |
| `LEVELS.kicad_sym` | Símbolos del proyecto (XIAO, RJ45, OLED, R, C) |
| `LEVELS.pretty/` | Huellas: `RJ45_8P8C_Shielded_Horizontal`, `XIAO-ESP32C3-SMD` |
| `fp-lib-table`, `sym-lib-table` | Registran las librerías del proyecto (no hay que instalar nada) |
| `RJ45_huella_1a1.pdf` | Huella del RJ45 a escala real para comprobar con el conector |

## Lista de materiales

| Ref | Valor | Huella | Notas |
|---|---|---|---|
| U1 | Seeed XIAO ESP32C3 | SMD (castellated) | USB-C al borde superior |
| J1 | RJ45 8P8C blindado 90° | THT | Tinaco |
| J2 | RJ45 8P8C blindado 90° | THT | **Opcional** (cisterna) |
| J3 | Zócalo hembra 1×4, 2.54 mm | THT | El OLED lleva header macho 1×4 |
| R1, R3 | 1 kΩ ¼ W | Axial, paso 7.62 | Divisor Echo (R3 opcional) |
| R2, R4 | 2 kΩ ¼ W | Axial, paso 7.62 | Divisor Echo (R4 opcional) |
| R5, R6 | 100 Ω ¼ W | Axial, paso 7.62 | Serie en Trig (R6 opcional) |
| C1, C3 | 220 µF 10 V (o más) | Radial D6.3, paso 2.5 | Respaldo de 5 V (C3 opcional) |
| C2, C4 | 100 nF cerámico | Disco, paso 5 | Desacoplo (C4 opcional) |
| H1–H4 | Barreno M3 | — | Para fijar a la caja impresa |

## Conexiones clave

| XIAO | Señal | Va a |
|---|---|---|
| D2 (GPIO4) | TRIG_T | R5 → J1 pin 1 |
| D3 (GPIO5) | ECHO_T | Divisor R1/R2 ← J1 pin 3 |
| D1 (GPIO3) | TRIG_C | R6 → J2 pin 1 |
| D10 (GPIO10) | ECHO_C | Divisor R3/R4 ← J2 pin 3 |
| D4 (GPIO6) | SDA | OLED SDA |
| D5 (GPIO7) | SCL | OLED SCK |
| 5V (VBUS) | +5V | J1/J2 pines 4-5, C1–C4 |
| 3V3 | +3V3 | OLED VCC |
| GND | GND | Todo lo demás + carcasa de los RJ45 |

RJ45 (T568B): 1-2 Trig/GND · 3-6 Echo/GND · 4-5 5V · 7-8 GND. Usa el mismo pinout en el conector del lado del sensor.

## Estado de la placa (v0.1)

- **Ruteada en 2 capas:** pistas de 0.4 mm para señales, 0.5 mm para +3V3 y 0.6 mm para +5V. Hay 6 vías de 0.8/0.4 mm.
- **Planos de GND** en ambas caras, con alivios térmicos. Hay dos zonas sin cobre:
  - bajo el XIAO en la cara superior (el módulo tiene pads expuestos por debajo);
  - en la zona de la antena U.FL, en ambas caras.
- **DRC (KiCad 7):** 0 cortos, 0 violaciones de separación, 0 pads sin conectar. Solo quedan 5 avisos de "alivio térmico con 1 rayo": son pads de GND que se conectan al plano con un solo rayo en lugar de dos. Están conectados y no afectan.
- **Conectividad:** la netlist que exporta KiCad del esquemático coincide red por red con la placa.

## Fabricación (`gerbers/`)

Comprime el contenido de `gerbers/` (o usa `LEVELS_gerbers.zip`) y súbelo a JLCPCB, PCBWay o similar:

- 2 capas, 1.6 mm, cobre 1 oz, acabado HASL (sin plomo si quieres)
- Medida 70 × 45 mm, separación y pista mínimas de 0.25 / 0.4 mm (cualquier fabricante lo cumple)
- Archivos de taladro separados: PTH (barrenos metalizados) y NPTH (postes del RJ45 y barrenos M3)

## Sobre el divisor (R2/R4 = 2 kΩ)

El divisor es 1 kΩ arriba y 2 kΩ abajo. Con 5 V en Echo da 3.33 V en el GPIO. Para la resistencia de abajo puedes usar:

- una de **2.0 kΩ** (serie E24, la ideal);
- una de **2.2 kΩ** (más fácil de conseguir: da 3.44 V, todavía seguro para el ESP32C3);
- **dos de 1 kΩ en serie**: eléctricamente es lo mismo, pero en esta placa solo hay lugar para una resistencia. Tendrías que soldarlas en vertical, una encima de la otra.

## Editar en KiCad

1. Abre `LEVELS.kicad_pro` (KiCad 7 o superior).
2. Si quieres las huellas oficiales de KiCad para R, C y el zócalo: **Herramientas → Actualizar huellas desde biblioteca**. Las incluidas tienen las mismas medidas.
3. Después de mover algo: **Editar → Rellenar todas las zonas (B)**, corre el DRC y vuelve a exportar los Gerbers.
4. `gen.py` regenera el esquemático y la PCB sin rutear, y `route.py` vuelve a rutear (requiere el Python de KiCad).

## Antes de mandar a fabricar — verifica

- **Huella del RJ45 (ya confirmada):** imprime `RJ45_huella_1a1.pdf` al 100 % (la barra debe medir 50 mm) y encima el conector. Las cotas verticales de la hoja de datos son ambiguas. Se interpretó así:
  - Origen en el punto medio de los postes de plástico.
  - Fila de pines pares (2-4-6-8) 3.43 mm detrás de los postes; fila impar (1-3-5-7) 1.78 mm más adelante.
  - Patitas del blindaje 4.32 mm detrás de los postes.
  - Boca del conector 5.5 mm delante de los postes (borde de la placa).

  Si algo no coincide, los valores están en el diccionario `RJ` de `gen.py` (o edita la huella en KiCad).
- **Huella del XIAO:** usa la posición de pads del footprint oficial de Seeed (OPL_Kicad_Library), solo los 14 pines laterales. Se omitieron los pads inferiores (batería/JTAG) para no dejar cobre expuesto debajo del módulo.
- **OLED:** el módulo queda sobre el zócalo y por encima de nada que sea alto. C1/C3 quedan fuera de su sombra. Revisa la altura del zócalo + header para la ventana de la tapa.
- Los botones B/R del XIAO quedan en el extremo de la antena: deja un orificio en la tapa para el reset.
