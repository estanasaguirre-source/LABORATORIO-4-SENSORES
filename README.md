# Laboratorio 4: Sensores capacitivos

Sistema de control de intensidad luminosa basado en un sensor capacitivo táctil tipo dial lineal, con acondicionamiento de señal y visualización en tiempo real mediante un ESP32.

**Universidad Militar Nueva Granada** · Ingeniería en Mecatrónica · Sensores y Laboratorio (V semestre)

## Descripción

La variación de capacitancia del sensor se convierte en frecuencia con un oscilador LM555, la señal se filtra y se transforma en tensión con un convertidor LM331. Esa tensión controla la potencia entregada a un bombillo de 9 VDC y es leída por el ESP32, que muestra el resultado en una pantalla LCD.

```
Sensor capacitivo → LM555 → Filtro pasa banda → LM331 → Etapa de potencia (bombillo)
                                                     └→ ESP32 (ADC) → LCD
```

## Contenido del repositorio

| Carpeta / archivo | Descripción |
|---|---|
| `esp32/codigo_esp32.ino` | Código Arduino del ESP32 (lectura del ADC y visualización en la LCD) |
| `matlab/caracterizacion_sensor.m` | Longitud vs. capacitancia, con regresión lineal, sensibilidad y errores |
| `matlab/linealizacion_sensor.m` | Tensión vs. capacitancia (subida), con regresión, R² y errores |
| `matlab/histeresis_sensor.m` | Curva de histéresis (subida y bajada), linealidad e histéresis máxima |
| `latex/` | Archivos fuente del informe |
| `README.md` | Este archivo |

## Diseño

**Sensor:** placas paralelas de hierro negro con acetato como dieléctrico. Capacitancia entre 50 pF y 123 pF.

**Oscilador LM555 (astable):**
`f = 1.44 / ((R1 + 2·R2)·C)` con R1 = 10 kΩ y R2 = 2 MΩ, lo que da una frecuencia entre 2.92 kHz y 7.18 kHz.

**Filtro pasa banda RC:**

| Etapa | Componentes | Frecuencia de corte |
|---|---|---|
| Pasa altas | C = 100 nF, R = 5.6 kΩ | 284.2 Hz |
| Pasa bajas | R = 22 kΩ, C = 470 pF | 15.39 kHz |

**Convertidor LM331 (frecuencia a tensión):**
`Vout = fin · 2.09 V · (RL/RS) · (Rt·Ct)` con Rt = 6.8 kΩ, Ct = 10 nF, RL = 100 kΩ y RS = 36.5 kΩ. Un filtro RC de salida (100 kΩ, 1 µF, fc ≈ 1.59 Hz) estabiliza la tensión.

**Etapa de potencia:** MOSFET que controla el bombillo de 9 VDC a partir de la tensión del LM331.

## Conexiones del ESP32

| Función | Pin |
|---|---|
| Entrada analógica (salida del LM331) | GPIO 34 |
| LCD I2C, SDA | GPIO 21 |
| LCD I2C, SCL | GPIO 22 |

La señal acondicionada debe mantenerse entre 0 V y 3.3 V. La dirección I2C de la LCD es `0x27` (si no funciona, probar `0x3F`).

## Cómo cargar el código

1. Instalar el soporte para ESP32 en el Arduino IDE.
2. Instalar la librería **LiquidCrystal_I2C** desde el Gestor de librerías.
3. Abrir `esp32/codigo_esp32.ino`, seleccionar la placa ESP32 y el puerto correspondiente.
4. Compilar y cargar.

El programa promedia 32 lecturas del ADC (12 bits, atenuación de 11 dB), calcula la tensión y muestra en la LCD el valor y un nivel según los umbrales `V_MIN`, `V_MED` y `V_MAX`. También envía los datos por el monitor serie a 115200 baudios.

## Scripts de MATLAB

Los scripts de `matlab/` no requieren toolboxes adicionales. Cada uno tiene los datos experimentales al inicio, calcula los resultados, grafica y los imprime en la ventana de comandos:

- **`caracterizacion_sensor.m`**: regresión de la capacitancia en función de la longitud (1 a 19 cm), con sensibilidad, R², error promedio y mayor error.
- **`linealizacion_sensor.m`**: regresión de la tensión en función de la capacitancia en el proceso de subida, con R² y errores.
- **`histeresis_sensor.m`**: compara subida y bajada, y calcula histéresis mínima, máxima y promedio, además de la linealidad de cada curva.

## Resultados principales

- Capacitancia medida: 50 pF a 121 pF (sensibilidad aproximada de 4.01 pF/cm).
- Tensión de salida medida: 2.059 V (70 pF) a 1.787 V (121 pF).
- Histéresis: 18.2 mV en promedio y 30 mV como máximo (6.3 % y 10.5 % del intervalo medido).

## Herramientas

Proteus (simulación de las etapas analógicas), Wokwi (simulación del ESP32 y la LCD), Arduino IDE y MATLAB.
