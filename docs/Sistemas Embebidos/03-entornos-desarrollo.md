---
title: "3. Entornos de Desarrollo: ¿Cómo usarlos?"
---
# 3. Entornos de Desarrollo: ¿Cómo usarlos?

Para pasar del hardware al uso real, necesitamos herramientas de desarrollo. La industria ha evolucionado más allá de los IDEs básicos.

## Arduino IDE
*(Menciona que es la opción oficial y fácil para empezar, pero que le faltan herramientas de ingeniería de software (autocompletado avanzado, gestión de dependencias compleja, integración nativa con Git).*

## PlatformIO (El estándar profesional)
*(Explica que PlatformIO, usado como extensión en VS Code, es la herramienta definitiva. Habla de cómo gestiona las librerías en un entorno aislado usando `platformio.ini` y soporta múltiples placas simultáneamente).*


[⬅ Anterior: Panorama](./02-hardware-esp32.md) | [Volver al Índice](./00-indice.md) | [Siguiente: Entornos de Desarrollo ➔](./04-mantenimiento-fiabilidad.md)

```mermaid
graph LR
    A[Código C/C++] --> B(PlatformIO Core)
    B -->|Librerías| C{Toolchain}
    C -->|Compilación| D[.bin / Firmware]
    D -->|Flasheo| E[ESP32]
