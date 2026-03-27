---
title: "1. Panorama Actual de los Sistemas Embebidos"
---
# 1. Panorama Actual de los Sistemas Embebidos

Por _Juan Antonio Bonillo García_

A la hora de hacer proyectos que involucren sensores, actuadores o cualquier tipo de hardware, suele ser necesario, si no imprescindible, utilizar un microcontrolador.

Los podemos encontrar en domótica, aviación, automoción, drones... sistemas de control en robótica, y en general en cualquier aplicación que requiera una gestión de hardware.

Esto se ha vuelto aún más relevante con la introducción del IoT (Internet Of Things); ahora incluso cosas que no tenían microcontrolador, como una bombilla, pueden tenerlo para una gestión remota.

Podríamos distinguir los tipos:

## 1.1 Microcontroladores a medida ASIC y FPGA: 
### 1.1.1 ASIC - Application-Specific Integrated Circuit
Este tipo de microcontroladores se diseñan a nivel de hardware para una misión específica. Son muy rápidos y baratos pues se enfocan exclusivamente en una función, pero tienen un coste de diseño y producción muy elevado, así que estarían, al menos en principio, fuera de las capacidades de una empresa pequeña. Se basan exclusivamente en puertas lógicas u otros circuitos, por ejemplo su uso está muy extendido para la conversión de señales analógicas a digitales; estos son llamados DSP (Digital Signal Processors).
### 1.1.2  FPGA (Field Programmable Gate Array)
Son un tipo de "procesador" que puede reprogramarse a nivel de conexiones, creando un circuito lógico con las puertas lógicas que necesitemos dentro del límite de sus conexiones, pero el costo de estos es muy elevado y es poco probable que valga la pena usarlos en producción.
### 1.1.3 Aplicación en la asignatura
En nuestros proyectos podríamos aprovechar estos microcontroladores para descargar a otros más genéricos de trabajos muy costosos o lentos. De esta manera, podríamos hacer más cosas con nuestro microcontrolador genérico o bien conseguir "tiempo real" (hablaremos de tiempo real más adelante). Si creemos que más adelante vamos a usar uno de estos, podemos empezar prototipando con un FPGA.
## 1.2 Microcontroladores de propósito general
Similares a una computadora estándar pero mucho más limitados (y baratos). Generalmente se utilizan para gestionar unas entradas físicas que se conocen como GPIO (General Purpose Input/Output). A su vez, también los podríamos dividir en dos según su arquitectura:
### 1.2.1 SBCs (Microordenadores con SO)  
Los SBC (Single Board Computer) tienen una arquitectura muy similar a una computadora de escritorio estándar pero en un SoC (System on Chip). Estos Microordenadores deben tener un sistema operativo como sus hermanos mayores y su potencia es considerable, sobre todo por su reducido precio (aunque no tan reducido como los siguientes que veremos). En esta categoría estaría la famosa Raspberry Pi y equivalentes. Estas máquinas permiten procesos múltiples y pueden ser el servidor que controle a otros microcontroladores menos potentes. Suelen contar con Wi-Fi, lo que los hace muy útiles para todo el entorno IoT.
### 1.2.2 MCUs (Microcontroladores Bare-Metal/RTOS)
Estos microcontroladores son muchísimo menos potentes, pero también muchísimo más baratos y con un uso de energía bajísimo, lo que los hace los candidatos ideales para ser microcontroladores de sensores, actuadores, cámaras... y cualquier otro hardware que no requiera una capacidad de cálculo muy alta. Habitualmente se trabaja con "Bare Metal", lo que quiere decir que el programa es un bucle que corre directamente en la máquina sin sistema operativo, aunque también existe un sistema operativo muy limitado llamado RTOS para aplicaciones más complejas que requieren una mejor gestión de procesos "simultáneos". Algunos ejemplos de estos son el Arduino o el ESP32 de Espressif.
### 1.2.3 GPUs y NPUs
Aunque no son microcontroladores en sí, estos pueden añadirse a los SBC para conseguir mejores prestaciones. En el caso de las GPUs (Graphics Processing Unit), permiten un cálculo paralelo masivo muy útil para visión artificial, y las NPUs (Neural Processing Unit) son ideales para redes neuronales con un gasto de energía mucho más contenido que una CPU o una GPU. Ejemplos: Google TPU y ARM Ethos.
## 1.3 Tendencias del Mercado
### 1.3.1 Edge Computing 
Cada vez más se está intentando evitar que los datos de cámaras y otros dispositivos que podrían violar la privacidad de las personas corran libremente por la red. De modo que en estos dispositivos se incluye un sistema embebido, un programa que mejora esta privacidad, por ejemplo, emborronando las caras de la gente o devolviendo exclusivamente el dato necesario (número de personas o detección de intrusos), pero sin devolver la imagen/datos en crudo.
### 1.3.2 TinyML (Machine Learning Embebido)
Relacionado con lo anterior, son modelos muy reducidos capaces de ejecutarse en microcontroladores y así permitir el Edge Computing con un costo muy reducido tanto de energía como de hardware, al margen de las ventajas ya comentadas del Edge Computing.
### 1.3.3 Hiperconectividad de bajo consumo (LPWAN)
Al ser dispositivos que utilizan muy poca energía, a veces incluso pilas de botón y similar, y la mayoría de las veces no requieren hacer grandes transmisiones de información, necesitan protocolos propios de comunicación de bajo consumo de energía o de largas distancias. Entre los más populares están:
#### 1.3.3.1 LoRaWAN
Es un protocolo de comunicación inalámbrica específico (no compatible con el Wi-Fi convencional IEEE 802.11) para entornos muy amplios (rural o infraestructura de una ciudad). Tiene una baja frecuencia (868 MHz), lo que le permite una mayor penetración en vegetación y edificios que el Wi-Fi convencional y un alcance de 5 a 15 km. Tiene tres tipos de consumo; el más eficiente es tan duradero (años) que la batería dura toda la vida útil del dispositivo.
#### 1.3.3.2 Bluetooth Low Energy (BLE)
Es un protocolo específico de Bluetooth pensado con la idea de que se pueda operar un sensor o similar durante años con una sola pila de botón. Está pensado de modo que el dispositivo pasa el 90% del tiempo dormido y se despierta solo para enviar ráfagas de datos.
#### 1.3.3.3 Zigbee y Thread 
Son redes en malla donde cada dispositivo se comporta a la vez como cliente y router; esto permite conectar naves enormes con una malla de dispositivos.
#### 1.3.3.4 ESPNOW
Es un protocolo diseñado por Espressif similar al Wi-Fi pero con mayor alcance, donde se eliminan los handshakes para conseguir una latencia muy baja, incluso inferior a 10 ms, ideal para aplicaciones de tiempo real.
## 1.4 Particularidades de los Microcontroladores
### 1.4.1 El uso de MQTT (Message Queuing Telemetry Transport)
En el mundo de los microcontroladores hay varios protocolos muy utilizados, este es el caso de MQTT, que es el estándar a la hora de comunicar sistemas embebidos. Es una cola en la que se garantiza que cada mensaje se entrega; una API HTTP no sería válida en un entorno donde la pérdida de un mensaje de un sensor o interruptor podría causar un error catastrófico.
### 1.4.2 El Tiempo Real
Para los microcontroladores hay un tema vital y es el "tiempo real". Esto no significa que algo se atienda lo más rápido posible, sino en un tiempo (siendo rápido) que sea siempre exactamente el mismo. Eso permite la sincronización de los sistemas y una situación reproducible cada vez. Por poner un ejemplo, yo necesito saber que cuando el sensor de presencia deja de detectar a un humano me va a contestar en 5ms exactamente, así sé cuándo cerrar la puerta automática; si lo hago antes o después podría golpear a una persona al cerrarse. Esto con un sistema operativo convencional es casi imposible pues se están computando varios programas al mismo tiempo, cambiando de ámbito cada vez, y este cambio no será siempre igual, lo que no los hace adecuados para estas tareas. Con "Bare Metal" o RTOS sí podemos garantizar un tiempo reproducible en cada ejecución, y esta es una de las grandes virtudes de los microcontroladores. 

[Volver al Índice](./00-indice.md) | [Siguiente: Hardware y ESP32 ➔](./02-hardware-esp32.md)