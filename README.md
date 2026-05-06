# Práctica 2B – Interrupción por Timer en ESP32

## Descripción

Este repositorio contiene la **Práctica 2B: Interrupción por Timer**, realizada para la asignatura **Procesadores Digitales**.

El objetivo de esta práctica es comprender el funcionamiento de las **interrupciones por temporizador** en el ESP32. Para ello, se configura un timer hardware que genera una interrupción cada segundo, sin utilizar `delay()` ni polling.

De esta forma, el procesador queda libre para ejecutar otras tareas mientras el temporizador se encarga de generar eventos periódicos.

## Autor

**Marco Carrasco Carmona**  
Fecha: **10 de marzo de 2026**

## Objetivo de la práctica

Comprender cómo funcionan las interrupciones periódicas mediante temporizadores hardware en el ESP32.

La práctica demuestra cómo ejecutar una acción cada segundo usando un timer, evitando bloquear el programa principal con funciones como `delay()`.

## Materiales utilizados

- Placa ESP32 DevKit
- Ordenador con VS Code
- PlatformIO
- Cable USB
- Monitor serial a 115200 baudios

No se necesita ningún componente externo, ya que la práctica se realiza únicamente con el ESP32 conectado por USB.

## Configuración del proyecto

Archivo `platformio.ini`:

```ini
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
upload_speed = 921600
```
Funcionamiento

El programa configura un temporizador hardware del ESP32 para generar una interrupción cada segundo.

Para ello:

Se inicializa el timer con timerBegin().
Se asocia una función ISR al temporizador mediante timerAttachInterrupt().
Se configura la alarma con timerAlarmWrite().
Se activa el temporizador con timerAlarmEnable().

Cada vez que se produce la interrupción, se ejecuta la función onTimer(), que incrementa una variable compartida llamada interruptCounter.

Después, en el loop(), se comprueba si ha ocurrido una interrupción. Si es así, se actualiza el contador total y se muestra un mensaje por el monitor serial.

Código principal
```cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;

hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR onTimer() {
  portENTER_CRITICAL_ISR(&timerMux);
  interruptCounter++;
  portEXIT_CRITICAL_ISR(&timerMux);
}

void setup() {
  Serial.begin(115200);

  // Configuración del timer
  timer = timerBegin(0, 80, true); // Timer 0, prescaler 80 -> 1 µs por tick
  timerAttachInterrupt(timer, &onTimer, true);
  timerAlarmWrite(timer, 1000000, true); // 1.000.000 µs = 1 segundo
  timerAlarmEnable(timer);
}

void loop() {
  if (interruptCounter > 0) {
    portENTER_CRITICAL(&timerMux);
    interruptCounter--;
    portEXIT_CRITICAL(&timerMux);

    totalInterruptCounter++;

    Serial.print("An interrupt has occurred. Total number: ");
    Serial.println(totalInterruptCounter);
  }
}
```
Explicación del timer

El ESP32 utiliza un reloj base de 80 MHz.

En esta práctica se configura el temporizador con un prescaler de 80, por lo que:

80 MHz / 80 = 1 MHz

Esto significa que cada tick del temporizador equivale a:

1 microsegundo

Por tanto, para generar una interrupción cada segundo se usa:

timerAlarmWrite(timer, 1000000, true);

Ya que:

1.000.000 microsegundos = 1 segundo

El último parámetro, true, activa el auto-reload, haciendo que la interrupción se repita automáticamente cada segundo.

Salida esperada por monitor serial
An interrupt has occurred. Total number: 1
An interrupt has occurred. Total number: 2
An interrupt has occurred. Total number: 3
An interrupt has occurred. Total number: 4
An interrupt has occurred. Total number: 5
Conceptos trabajados
Interrupciones por timer
Temporizadores hardware del ESP32
Uso de timerBegin()
Uso de timerAttachInterrupt()
Uso de timerAlarmWrite()
Uso de timerAlarmEnable()
Rutinas de servicio de interrupción, ISR
Uso de IRAM_ATTR
Variables volatile
Secciones críticas con portMUX
Ventajas frente al uso de delay() y polling
Resultados

La práctica funcionó correctamente.
El ESP32 generó una interrupción cada segundo de forma precisa, mostrando por el monitor serial el número total de interrupciones ocurridas.

El contador totalInterruptCounter aumentó indefinidamente mientras el programa estaba en ejecución.

Observaciones
La interrupción se produce cada segundo con precisión de hardware.
No se utiliza delay(), por lo que el procesador no queda bloqueado.
El loop() queda disponible para ejecutar otras tareas.
El uso de volatile es necesario porque interruptCounter puede modificarse dentro de la ISR.
Las secciones críticas protegen el acceso a variables compartidas entre la ISR y el programa principal.
Conclusión

Esta práctica demuestra que las interrupciones por temporizador son una forma eficiente y precisa de ejecutar tareas periódicas en el ESP32.

A diferencia del uso de delay(), el temporizador permite que el procesador siga disponible para otras operaciones mientras las interrupciones se generan automáticamente cada segundo.

Esta técnica es muy útil en proyectos reales donde se necesita controlar tiempos con precisión, como lectura periódica de sensores, generación de señales, comunicación o control de procesos
