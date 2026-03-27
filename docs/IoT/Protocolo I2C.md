# Protocolo I2C y cómo conectar diferentes dispositivos idénticos
El protocolo I2C (Inter-Integrated Circuit) es un protocolo estándar de bus de datos serie desarrollado por Philips Semiconductors en 1982. El protocolo fue diseñado con el objetivo de poder comunicar de una manera sencilla diferentes circuitos integrados ubicados dentro de una misma placa, como ocurría internamente en los televisores de la época. El protocolo se volvió muy popular debido a su gran versatilidad y bajo coste, ya que solo requiere 2 cables para realizar la comunicación. Con los años se convirtió en un estándar mundial, sobre todo para conectar microcontroladores, periféricos y todo tipo de pantallas y sensores.
![image info](/docs/images/IoT/I2C/esquema_i2c.png)

## Características del protocolo
Las principales características del protocolo I2C son las siguientes:
- Es un protocolo de comunicación en serie en el que se transmite información bit a bit.
- Es un protocolo bidireccional half duplex por lo que se pueden efectuar las comunicaciones en las dos direcciones, pero solo una parte puede estar hablando a la vez.
- El protocolo es multimaestro y multiesclavo lo que permite que se comuniquen diferentes dispositivos. Cuando existen varios maestros, solo uno ejerce de maestro y el resto actúan como esclavos.
- Solo utiliza 2 cables. Una línea para los datos llamada SDA y otra para el reloj llamada SCL.
- Es síncrono, por lo que ambos dispositivos se sincronizan utilizando el tick de reloj que proporciona el maestro por el canal SCL. Este tick de reloj indica la velocidad a la que se van a transmitir los datos. Existen diferentes especificaciones de velocidades en el estándard I2C:

    | Modo | Velocidad |
    |------|-----------|
    | Standard-mode (Sm) | Hasta 100 kbit/s |
    | Fast-mode (Fm) | Hasta 400 kbit/s |
    | Fast-mode Plus (Fm+) | Hasta 1 Mbit/s |
    | High-speed mode (Hs-mode) | Hasta 3.4 Mbit/s |

- Ambas líneas utilizan resistencias de pull-up para mantener el bus en un estado lógico alto cuando se encuentra inactivo.


## Comunicación
En este protocolo, la comunicación siempre es iniciada por el maestro. A continuación, vamos a ver cómo funciona la secuencia de comunicación entre los maestros y los esclavos. Toda la comunicación se realiza por SDA (Serial Data); SCL (Serial Clock) solo tiene los pulsos de reloj generados por el maestro.
1. El maestro cambia la señal en SDA de alto a bajo cuando SCL está en alto. A partir de aquí se inicia la señal de reloj en SCL (por el maestro). 
2. El maestro envía la dirección del esclavo con el que se quiere comunicar, seguido por un bit de lectura/escritura que ahora estará siempre en 0.
3. El esclavo debe responder con un bit de confirmación (ACK) en nivel bajo. (A partir de aquí el protocolo cambia para leer y escribir):
![image info](/docs/images/IoT/I2C/inicio_conexión.png)
4. Escritura:
    1. El maestro envía el registro de 8 bits a modificar.
    2. El esclavo contesta con un ACK.
    3. El maestro envía el dato que se quiere escribir de 8 bits en el registro anterior.
    4. El esclavo contesta con un ACK.
    5. Si el maestro envía otro dato, se repetirá la secuencia de los 2 pasos anteriores, pero escribiendo los datos en el siguiente registro.
    6. La conexión acaba cuando el maestro cambia la señal del SDA de bajo a alto cuando el SCL está en alto.
    ![image info](/docs/images/IoT/I2C/escritura.png)
5. Lectura:
    1. El maestro envía el registro de 8 bits a leer.
    2. El esclavo contesta con un ACK.
    3. El maestro cambia la señal en SDA de alto a bajo cuando SCL está en alto. (Otra señal de inicio).
    4. El maestro vuelve a enviar la dirección del esclavo, pero seguido por el bit de lectura/escritura en alto (1).
    5. El esclavo contesta con un ACK.
    6. El esclavo envía los datos del registro indicado anteriormente.
    7. El maestro contesta con un ACK.
    8. Se repiten estos 2 últimos pasos, pero enviando los datos del siguiente registro hasta que el maestro envíe una señal de parada (SDA de bajo a alto cuando SCL está en alto).
    ![image info](/docs/images/IoT/I2C/lectura.png)

## Cómo se asignan las direcciones
Para que el maestro pueda saber exactamente con qué esclavo está hablando a través del bus, cada esclavo cuenta con una dirección única que lo identifica. Esta dirección suele ser de 7 bits, lo que, en teoría, permite tener hasta 128 dispositivos diferenciados en el mismo bus. La dirección de cada esclavo viene predefinida por el fabricante en el propio chip. En algunos casos, el fabricante permite modificar los últimos pines de la dirección conectando algún pin físico a voltaje o a tierra. Estos cambios suelen ser mínimos.

El protocolo no cuenta con ningún mecanismo para identificar dispositivos, sino que el propio maestro tiene que saber de antemano las direcciones de los esclavos para poder comunicarse con ellos.

## El problema de asignar direcciones en múltiples sensores idénticos 
Dado que las direcciones vienen establecidas en su fabricación, surge un problema cuando tratamos de utilizar muchos sensores idénticos, ya que todos tienen la misma dirección. Esto genera un conflicto de direcciones si los conectamos todos al bus, porque cuando el maestro trate de comunicarse con uno, todos responderán, ya que tienen el mismo identificador. En el caso de que dispongamos, por ejemplo, de dos pantallas OLED, ambas mostrarán el mismo contenido sin poder distinguir una de otra.
![image info](/docs/images/IoT/I2C/colision_direcciones.png)

## Solución utilizando el multiplexor TCA9548A 
Para solucionar este problema, surgen los multiplexores TCA9548A que nos permiten conectar hasta 8 dispositivos en una misma línea I2C. Este circuito, que actúa como un conmutador de tráfico inteligente, está conectado al bus principal I2C con nuestro dispositivo maestro. El multiplexor tiene una dirección I2C asignada (puede ser 0x70 o 0x77 si puenteamos 2 pines) por lo que el dispositivo maestro se puede comunicar de manera normal con él. Cuando el dispositivo maestro quiere consultar cualquiera de los sensores, le envía un único byte de configuración al multiplexor para indicarle con qué dispositivo quiere comunicarse. El circuito TCA9548A activa ese canal de comunicación y permite al maestro transmitir información directamente con ese dispositivo utilizando la dirección del multiplexor en el canal I2C.
![image info](/docs/images/IoT/I2C/diagrama%20conexiones.png)

## Conclusiones 
El uso de multiplexores soluciona el problema de tener diferentes sensores con la misma dirección en el mismo bus I2C, lo que nos permite disponer de una cantidad enorme de sensores. Sin embargo, todo esto no tiene coste 0, sino que añade una sobrecarga de tráfico al sistema que añade latencia, ya que cada comunicación con cada sensor dentro del multiplexor requiere diferentes comandos de comunicación entre el maestro y el multiplexor para intercambiar entre los diferentes canales. A esto se le suma el aumento de dispositivos en el bus I2C lo que ralentizará el sistema, ya que, cuántos más dispositivos existan en el bus comunicándose al mismo tiempo, menor será la velocidad de todas las comunicaciones.