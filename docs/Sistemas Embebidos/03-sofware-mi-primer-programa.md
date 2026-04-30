---
title: "3. Programación Avanzada: Del Bucle Secuencial al RTOS"
---
# 3. Programación Avanzada: Del Bucle Secuencial al RTOS

En este capítulo aprenderemos a programar con una mentalidad de ingeniería. Utilizaremos un reto práctico para comparar cómo trabaja un sistema básico (estilo Arduino Uno) frente a un sistema profesional multitarea (ESP32 con FreeRTOS).

## Índice de contenidos
1. [El Reto: El Blink Adaptativo](#1-el-reto-el-blink-adaptativo)
2. [Aproximación Secuencial (Bucle Tradicional)](#2-aproximacion-secuencial-bucle-tradicional)
3. [Aproximación Profesional (FreeRTOS)](#3-aproximacion-profesional-freertos)
4. [Análisis de Diferencias y Ventajas](#4-analisis-de-diferencias-y-ventajas)

---

## 1. El Reto: El Blink Adaptativo
Queremos diseñar un sistema con un LED y un pulsador. La velocidad de parpadeo del LED debe aumentar con cada pulsación del botón:
* **0 pulsaciones:** LED apagado.
* **1 a 5 pulsaciones:** Velocidad incremental (cada vez más rápido).
* **6ª pulsación:** El ciclo vuelve a 0 y el LED se apaga.

---

## 2. Aproximación Secuencial (Bucle Tradicional)

En el modelo tradicional de Arduino, el procesador ejecuta una instrucción tras otra en un hilo único. 

### El problema del "Bloqueo"
Si usamos la función `delay()`, el procesador se detiene por completo. Si pulsas el botón mientras el LED está en medio de un `delay(1000)`, la pulsación **se pierde** porque el procesador está "ciego". Para solucionarlo, debemos usar una técnica llamada **Polling** basada en el tiempo transcurrido (`millis()`).

### Código en Loop (Simulación de paralelismo)

```cpp
#include <Arduino.h>

const int LED_PIN = 2;
const int BTN_PIN = 4;

int nivelVelocidad = 0; 
unsigned long ultimoParpadeo = 0;
unsigned long ultimaPulsacion = 0;
bool estadoLed = false;

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BTN_PIN, INPUT_PULLUP); // Resistencia interna activa
}

void loop() {
  // 1. ESCANEO DEL BOTÓN (Debouncing manual)
  if (digitalRead(BTN_PIN) == LOW && (millis() - ultimaPulsacion > 200)) {
    nivelVelocidad = (nivelVelocidad + 1) % 6; 
    ultimaPulsacion = millis();
  }

  // 2. LÓGICA DEL LED (Sin usar delay)
  if (nivelVelocidad > 0) {
    int intervalo = 1000 / nivelVelocidad; // A más nivel, menos tiempo (más rápido)
    if (millis() - ultimoParpadeo >= intervalo) {
      estadoLed = !estadoLed;
      digitalWrite(LED_PIN, estadoLed);
      ultimoParpadeo = millis();
    }
  } else {
    digitalWrite(LED_PIN, LOW);
  }
}
```
### Desglose del codigo secuencial

#### 2.1 Sección de cabecera y definición de constantes
  Casi idéntico a cualquier programa de C++, tenemos los includes de las librerías que necesitaremos (en algunos casos serán las propias de algún sensor o actuador). Para no usar "números mágicos" a lo largo del programa, definiremos aquí las constantes:

```cpp

#include <Arduino.h>

const int LED_PIN = 2;
const int BTN_PIN = 4;

int nivelVelocidad = 0; 
unsigned long ultimoParpadeo = 0;
unsigned long ultimaPulsacion = 0;
bool estadoLed = false;
```
#### 2.2 Sección de configuración (setup)
En esta sección definiremos todo lo necesario para que los sensores y actuadores funcionen correctamente. Esta sección solo se ejecutará una vez cuando se conecte o reinicie el dispositivo:

```cpp
void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BTN_PIN, INPUT_PULLUP); // Resistencia interna activa
}
```
#### 2.3 El motor del programa (loop)
Esta es la parte del código que se repite una y otra vez mientras el microcontrolador esté encendido. Podemos llamar a otras funciones, pero el código se ejecutará de manera estrictamente secuencial:

```cpp
void loop() {
  // 1. ESCANEO DEL BOTÓN (Debouncing manual)
  if (digitalRead(BTN_PIN) == LOW && (millis() - ultimaPulsacion > 200)) {
    nivelVelocidad = (nivelVelocidad + 1) % 6; 
    ultimaPulsacion = millis();
  }

  // 2. LÓGICA DEL LED (Sin usar delay)
  if (nivelVelocidad > 0) {
    int intervalo = 1000 / nivelVelocidad; // A más nivel, menos tiempo (más rápido)
    if (millis() - ultimoParpadeo >= intervalo) {
      estadoLed = !estadoLed;
      digitalWrite(LED_PIN, estadoLed);
      ultimoParpadeo = millis();
    }
  } else {
    digitalWrite(LED_PIN, LOW);
  }
}
```

## 3. Aproximación Profesional (FreeRTOS)

En un ESP32, no estamos limitados a un solo hilo de ejecución. Gracias a FreeRTOS (Real-Time Operating System), podemos crear tareas independientes que se ejecutan "simultáneamente".
¿Cómo funciona?

Creamos dos tareas que se reparten el trabajo:

    Tarea de Control: Solo vigila el botón. Si se pulsa, actualiza una variable.

    Tarea de Actuación: Solo hace parpadear el LED a la velocidad que dicte la variable.

Código Multitarea (Mentalidad de Ingeniería)

En este código observaremos que la función loop() está prácticamente vacía. Con RTOS, delegamos el flujo del programa en la creación de tareas. Estas pueden ser de un solo uso (tareas que esperan a un evento, se ejecutan una sola vez y se destruyen) o, como es habitual, contener su propio bucle infinito individual.

```cpp
#include <Arduino.h>

const int LED_PIN = 2;
const int BTN_PIN = 4;

// Variable volátil: indica al compilador que puede cambiar en cualquier momento
volatile int nivelGlobal = 0; 

// --- TAREA 1: GESTIÓN DEL BOTÓN ---
void tareaBoton(void *pvParameters) {
  for (;;) { // Bucle infinito de la tarea
    if (digitalRead(BTN_PIN) == LOW) {
      nivelGlobal = (nivelGlobal + 1) % 6;
      vTaskDelay(pdMS_TO_TICKS(250)); // Pausa la tarea, NO el procesador
    }
    vTaskDelay(pdMS_TO_TICKS(10)); // Cede el tiempo sobrante al sistema
  }
}

// --- TAREA 2: GESTIÓN DEL LED ---
void tareaLed(void *pvParameters) {
  for (;;) {
    if (nivelGlobal > 0) {
      int tiempoMs = 1000 / nivelGlobal;
      digitalWrite(LED_PIN, HIGH);
      vTaskDelay(pdMS_TO_TICKS(tiempoMs / 2));
      digitalWrite(LED_PIN, LOW);
      vTaskDelay(pdMS_TO_TICKS(tiempoMs / 2));
    } else {
      digitalWrite(LED_PIN, LOW);
      vTaskDelay(pdMS_TO_TICKS(100)); // Espera eficiente si está apagado
    }
  }
}

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BTN_PIN, INPUT_PULLUP);

  // CREACIÓN DE TAREAS (Asignadas al programador de FreeRTOS)
  xTaskCreate(tareaBoton, "LecturaBoton", 2048, NULL, 1, NULL);
  xTaskCreate(tareaLed, "ControlLed", 2048, NULL, 1, NULL);
}

void loop() {
  // El bucle principal queda libre para otras funciones o puede eliminarse
  vTaskDelete(NULL); 
}
```
## 4. Análisis de Diferencias y Ventajas
¿Por qué el enfoque RTOS es el estándar industrial?

    Determinismo: En el código loop(), si mañana añadimos una función que tarda mucho tiempo en ejecutarse (como conectar a una red Wi-Fi), el botón dejará de responder correctamente. En RTOS, la tarea del botón tiene su propio tiempo garantizado por el sistema operativo.

    Eficiencia Energética: Mientras una tarea usa vTaskDelay(), el procesador entra en estados de bajo consumo automáticamente. El delay() tradicional de Arduino mantiene el procesador al 100% de potencia "quemando" ciclos.

    Escalabilidad: Imagina que ahora quieres añadir una pantalla OLED que muestre la velocidad actual. En el modelo secuencial, tendrías que reescribir e integrar todo en el loop. En RTOS, simplemente creas una tercera tarea llamada tareaPantalla y el sistema operativo se encarga de que conviva con las demás.

Conclusión Técnica

Para proyectos simples de aprendizaje, el bucle secuencial es suficiente. Sin embargo, para cualquier dispositivo que deba gestionar comunicaciones (Wi-Fi/Bluetooth) y entradas físicas al mismo tiempo, el uso de un RTOS es obligatorio para garantizar la fiabilidad y la respuesta en tiempo real.

[⬅ Anterior: Hardware y ESP32](./02-hardware-esp32.md) | [Volver al Índice](./00-indice.md) | [Siguiente: Entornos de desarrollo ➔](./04-entornos-desarrollo.md)