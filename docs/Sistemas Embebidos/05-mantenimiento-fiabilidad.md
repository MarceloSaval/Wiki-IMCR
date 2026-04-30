---
title: "5. Mantenimiento y Fiabilidad"
---
# 5. El Enfoque IMCR: Mantenimiento en Producción

El diseño de un sistema embebido profesional no termina cuando el código compila sin errores en nuestro ordenador. Una vez que el dispositivo (como nuestro ESP32) es desplegado en su entorno final de producción —por ejemplo, instalado en el techo a 10 metros de altura en una nave industrial o en el interior de maquinaria pesada—, el acceso físico al mismo se vuelve logísticamente complejo y económicamente muy costoso. Por ello, la ingeniería de software embebido exige aplicar un enfoque de diseño tolerante a fallos, donde el propio sistema sea capaz de recuperarse de errores imprevistos y actualizarse sin requerir el desplazamiento de un operario o técnico.

## 5.1 Mantenimiento Autónomo: Watchdog Timer (WDT)



El **Watchdog Timer** (o Perro Guardián) es un mecanismo de seguridad fundamental basado en hardware. Consiste en un temporizador físico, independiente del núcleo principal del procesador, que realiza una cuenta regresiva de forma continua. 

En condiciones normales de funcionamiento, el software de nuestro ESP32 debe reiniciar este contador periódicamente antes de que llegue a cero (una acción conocida en la industria como *"alimentar al perro"* o *"kick the dog"*). Sin embargo, si el software principal se queda bloqueado (por un bucle infinito mal programado, un fallo en la conexión Wi-Fi que deja el hilo colgado, o un pico electromagnético transitorio), el procesador dejará de alimentar al Watchdog. 

Cuando el contador del WDT llega a cero, el hardware asume que el sistema ha sufrido un fallo crítico y provoca automáticamente un reinicio eléctrico forzado (Reset). Esta estrategia de **mantenimiento correctivo autónomo** garantiza que un dispositivo "colgado" vuelva a estar operativo en cuestión de segundos, sin ninguna intervención humana.

## 5.2 Mantenimiento Evolutivo: Actualizaciones OTA

El **mantenimiento evolutivo** consiste en añadir nuevas funcionalidades o corregir vulnerabilidades de seguridad (parches) una vez que el dispositivo ya está operando en la planta. En el ecosistema tradicional (como el Arduino clásico), esto exigía desmontar el equipo y conectar un cable directamente al puerto serie del microcontrolador.

Gracias a la conectividad Wi-Fi del ESP32 y a su generosa memoria Flash, podemos implementar actualizaciones **OTA (Over-The-Air)**. Esta tecnología permite compilar un nuevo firmware desde nuestro entorno (como PlatformIO) y enviarlo de forma inalámbrica y cifrada a través de la red local o desde un servidor en la nube. 

El proceso es altamente seguro:
* El ESP32 recibe el archivo y lo escribe en una partición secundaria de su memoria.
* Una vez completada y verificada la descarga, el dispositivo se reinicia apuntando a esta nueva partición.
* Si el nuevo código es defectuoso, el sistema puede revertir los cambios volviendo a la partición anterior.

Esto permite actualizar flotas enteras de miles de dispositivos distribuidos geográficamente con un solo clic, reduciendo drásticamente los costes de mantenimiento.

## 5.3 Fiabilidad del Sistema

Desde la ingeniería de mantenimiento, asumiendo una tasa de fallos constante ($\lambda$), la fiabilidad $R(t)$ del dispositivo (la probabilidad de que funcione sin fallos durante un tiempo $t$) se calcula con la distribución exponencial:

$$R(t) = e^{-\lambda t}$$

Implementar estrategias como el OTA y el WDT en nuestro código no reduce los fallos físicos de hardware por fatiga térmica o electrónica, pero **reduce drásticamente el Tiempo Medio de Reparación (MTTR)** ante bloqueos de software y errores de lógica. Al bajar el MTTR a valores cercanos a cero (gracias al reinicio automático del WDT), aumentamos radicalmente la disponibilidad y la fiabilidad global de nuestra red de sistemas embebidos en el ámbito del IoT industrial.

---

[⬅ Anterior: Entornos](./04-entornos-desarrollo.md) | [Volver al Índice](./00-index.md)