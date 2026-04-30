---
title: "4. Entornos de Desarrollo: De Prototipo a Producción"
---
# 4. Entornos de Desarrollo: De Prototipo a Producción

En este capítulo analizaremos las herramientas que nos permiten dar vida al hardware. No se trata solo de escribir código, sino de cómo gestionamos la complejidad de un sistema profesional.

## Índice de contenidos
1. [Comparativa: Arduino IDE vs. PlatformIO](#comparativa-arduino-ide-vs-platformio)
2. [Beneficios y Desventajas](#beneficios-y-desventajas)
3. [Mini-Tutorial: Mi primer "Blink"](#mini-tutorial-mi-primer-blink)
    * [Caso A: Arduino IDE](#caso-a-arduino-ide)
    * [Caso B: PlatformIO (VS Code)](#caso-b-platformio-vs-code)
4. [Conclusiones: ¿Cuál elegir?](#conclusiones-cual-elegir)

---

## 1. Comparativa: Arduino IDE vs. PlatformIO

Para entender la diferencia, debemos ver cómo gestionan un proyecto. Mientras uno es un "bloc de notas con esteroides", el otro es un sistema de gestión de ingeniería.

| Característica | Arduino IDE 2.0 | PlatformIO (en VS Code) |
| :--- | :--- | :--- |
| **Instalación** | Muy simple (Standalone) | Extensión de VS Code |
| **Gestión de Librerías** | Global (Para todos los proyectos) | Local (Aislada por proyecto) |
| **Autocompletado** | Básico | Avanzado (IntelliSense) |
| **Control de Versiones** | Manual / Externo | Integración total con Git |
| **Debugging** | Muy limitado (Serial) | Avanzado (Puntos de interrupción) |
| **Soporte Multijunta** | Una a la vez | Varias placas simultáneas |



---

## 2. Beneficios y Desventajas

### 2.1 Arduino IDE
* **Beneficios:** Ideal para principiantes. No requiere configurar archivos complejos. Si quieres encender un LED en 30 segundos, es tu herramienta.
* **Desventajas:** Inviable para proyectos grandes. Si actualizas una librería para un sensor nuevo, podrías "romper" sin querer el código de un proyecto antiguo que usaba la versión anterior de esa misma librería.

### 2.2 PlatformIO
* **Beneficios:** Profesionalismo total. Gracias al archivo `platformio.ini`, el proyecto es "autocontenido". Puedes retomar un código dentro de 5 años y funcionará porque las librerías están fijadas en ese proyecto.
* **Desventajas:** Curva de aprendizaje inicial más alta. Requiere entender conceptos como rutas de archivos y archivos de configuración.

---

## 3. Mini-Tutorial: Mi primer "Blink"

Vamos a ver cómo haríamos el clásico programa que hace parpadear un LED.

### Caso A: Arduino IDE
1.  **Conecta** tu placa.
2.  **Selecciona** la placa en `Herramientas > Placa`.
3.  **Escribe** el código directamente en la ventana blanca:
    ```cpp
    void setup() {
      pinMode(2, OUTPUT);
    }
    void loop() {
      digitalWrite(2, HIGH);
      delay(1000);
      digitalWrite(2, LOW);
      delay(1000);
    }
    ```
4.  Pulsa la **Flecha (Subir)** y listo.

### Caso B: PlatformIO (VS Code)
1.  Crea un **New Project** en el Home de PlatformIO.
2.  **Configura**: Selecciona tu placa (ej. ESP32 Dev Module) y Framework (Arduino).
3.  **Gestión de archivos**: El código no se escribe en cualquier lado; debes abrir la carpeta `src` y buscar el archivo `main.cpp`.
4.  **Añade la cabecera**: A diferencia de Arduino, aquí **debes** incluir la librería al principio:
    ```cpp
    #include <Arduino.h> // Obligatorio en PlatformIO
    
    void setup() {
      pinMode(2, OUTPUT);
    }
    ...
    ```
5.  **Compila y Sube**: Usa los iconos de la "Flecha" o el "Check" en la barra azul inferior de VS Code.

---

## 4. Conclusiones

* **Usa Arduino IDE si:** Estás aprendiendo las bases de la programación, vas a hacer un proyecto de un solo archivo o necesitas probar un sensor rápidamente.
* **Usa PlatformIO si:** Estás desarrollando un producto que vas a vender o entregar, si trabajas en equipo (Git) o si tu proyecto usa más de 2 librerías externas. **Es la herramienta que te define como profesional.**

[⬅ Anterior: Software: Mi primer programa en un microcontrolador](./03-software-mi-primer-programa.md) | [Volver al Índice](./00-indice.md) | [Siguiente: Mantenimiento y Fiabilidad ➔](./05-mantenimiento-fiabilidad.md)