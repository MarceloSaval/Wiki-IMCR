---
title: "2. Hardware: Arduino vs ESP32 en Producción"
---
# 2. Hardware: Arduino vs. ESP32 en Producción

Por _Juan Antonio Bonillo García_

A la hora de diseñar un sistema embebido, la elección del microcontrolador dictamina la viabilidad técnica y económica del proyecto:

## 2.1 Ecosistema Arduino (ej. Arduino UNO)
Arduino fue el gran impulsor de los microcontroladores modernos. Su orientación inicial era que los aficionados se introdujeran en el mundo de la electrónica, el manejo de robótica y los sensores, pero no tenía el foco en el entorno de producción real. Las limitaciones de Arduino son evidentes: hasta hace poco no tenía conectividad Wi-Fi ni Bluetooth, y su precio era elevado. A nivel didáctico es muy útil, pero ha quedado eclipsado por una competencia que ha llevado el MCU un paso más allá en todos los sentidos.

<figure markdown="Diagrama funcional del ATmega328P (Arduino Uno) a 16 MHz">
    <img src="../../images/Sistemas Embebidos/ATmega328P-functional-diagram.png">
</figure>

Aquí podemos ver cómo el ATmega328P carecía de módulos físicos para gestionar el Wi-Fi, la seguridad o el bajo consumo. De hecho, actualmente se venden placas de desarrollo Arduino con el chip ESP32 integrado.

## 2.2 Familia ESP32 (Espressif)
Espressif, con su ESP32 y todos los modelos sucesivos, llevó el MCU a un precio y unas prestaciones que ya permitían usarlo en placas de producción industrial, no solo en placas didácticas de desarrollo. Algunos de los puntos clave que permitieron esto fueron:

<figure markdown="Diagrama funcional del Tensilica Xtensa LX6 (ESP32) a 240 MHz">
    <img src="../../images/Sistemas Embebidos/ESP32-functional-block-diagram.png">
</figure>

En este diagrama podemos observar los múltiples módulos del LX6 dedicados a funciones avanzadas como encriptación por hardware, Wi-Fi o ultra bajo consumo.

### 2.2.1 Doble núcleo
Probablemente implementado para poder seguir garantizando el tiempo real a pesar de usar Wi-Fi o Bluetooth (inicialmente no se podían usar ambos a la vez porque las librerías saturaban la memoria). Posteriormente se le dio acceso a este segundo procesador al programador, lo cual abría un abanico de verdadero paralelismo que el núcleo simple de Arduino no permitía.

### 2.2.2 Módulo Wi-Fi y Bluetooth
Como hemos comentado antes, aunque en sus inicios tenía limitaciones de memoria para usar ambos simultáneamente, ofrecía conectividad nativa tanto Bluetooth como Wi-Fi, con módulos de radio físicos específicos para ocuparse de esta tarea de manera eficiente.

### 2.2.3 Módulo de encriptación
Cuenta con un módulo específico dedicado a la encriptación por hardware (AES, SHA, RSA). Esto es vital en un entorno IoT donde el aparato sale directamente a Internet transmitiendo datos que podrían ser sensibles.

### 2.2.4 Coprocesador de ultra bajo consumo (ULP)
Esta característica le ha permitido operar con baterías durante meses. Este procesador extra, de muy baja velocidad y consumo, se encarga de "despertar" a los núcleos principales cuando se dan ciertas circunstancias (un temporizador, un cambio de estado en un pin GPIO, etc.). Esto permite que el procesador pase de consumir miliamperios (mA) a apenas unos pocos microamperios (µA) en modo *Deep Sleep*.

### Resumen Técnico Comparativo

| Característica | Arduino UNO (ATmega328P) | ESP32 (Xtensa Dual-Core) |
| :--- | :--- | :--- |
| **Arquitectura** | 8-bits (Un núcleo) | 32-bits (Doble núcleo) |
| **Velocidad de Reloj** | 16 MHz | 160 - 240 MHz |
| **Memoria RAM** | 2 KB | 520 KB |
| **Conectividad** | Ninguna nativa | Wi-Fi b/g/n + Bluetooth (Classic/BLE) |
| **Seguridad Hardware**| No | Criptografía AES, SHA, RSA |
| **Precio aprox. (Chip)**| ~2.50 € | ~1.50 € |


## 2.3 Microordenadores (Raspberry Pi y similares)
Cuando necesitamos más potencia para TinyML o Edge Computing y el tiempo real estricto no es el factor más importante, puede que sea mejor dar el salto y utilizar microordenadores como la Raspberry Pi o similares, apoyados incluso con hardware dedicado como una TPU de Google.

[⬅ Anterior: Panorama](./01-panorama-actual.md) | [Volver al Índice](./00-indice.md) | [Siguiente: Entornos de Desarrollo ➔](./03-entornos-desarrollo.md)