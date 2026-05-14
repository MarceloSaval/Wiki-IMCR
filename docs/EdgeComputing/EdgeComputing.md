# Edge Computing


## ¿Qué es?

Es un **paradigma de computación distribuida** que acerca computación y almacenamiento de datos a la ubicación en la que se necesita para mejorar los tiempos de respuesta y ahorrar ancho de banda. Es decir, el sistema crítico está en el local (sensores, servidores locales, equipos, etc).

### Caso tradicional

```text
Dispositivo → Internet → Nube → Procesamiento → Respuesta
```

### Caso de modelo Edge

```text
Dispositivo → Nodo Edge local → Procesamiento rápido → Respuesta
                              ↓
                         Nube, si hace falta
```

Para aclarar la definición, citamos a Cisco, una empresa relacionada a seguridad de redes, "es una arquitectura de TI distribuida que procesa datos cerca de su fuente utilizando tecnologías locales de computación, almacenamiento, redes y seguridad. Al estar más cerca de donde se generan los datos, el edge computing reduce la latencia, mejora la capacidad de respuesta en tiempo real y reduce los costos de ancho de banda. Esto es especialmente valioso para aplicaciones que requieren una toma de decisiones instantánea, como la automatización industrial, el comercio minorista inteligente y la telemedicina.""

## ¿Para qué sirve?
El **Edge Computing** sirve para mejorar sistemas donde enviar todos los datos a la nube no es eficiente o no es suficientemente rápido.

Sus usos principales son:

| Uso | Explicación |
|---|---|
| Reducir latencia | La decisión se toma cerca del dispositivo. |
| Ahorrar ancho de banda | No se mandan todos los datos brutos a la nube. |
| Mejorar privacidad | Parte de los datos se queda en local. |
| Funcionar con mala o sin conexión | El sistema puede seguir actuando aunque Internet falle. |
| Procesar en tiempo real | Útil en industria, coches, sanidad, cámaras o IoT. |

## Usos reales (opcional)


## Tecnologías del Edge

Presentamos algunas tecnologías usadas en el mundo de edge computing, como WASM, K3s, ZTE y TinyML.

### TinyML

TinyML permite ejecutar modelos de inteligencia artificial en dispositivos muy pequeños, como sensores o microcontroladores. Por ejemplo, en el caso de la pulsera FitBit Charge 5 lo usa para obtener los datos del estrés y dar una recomendación más ersonalizada para tartarlo. O en el caso más complejo, el autopiloto de Tesla (2023), con los chips HW4 de Samsung de 8gb de RAM y 128gb ROM y con 20 núcleos de ARM exynos, que sus funciones son visualizar las imágenes y procesarlas, tener mayor eficiencia y la autoconducción más avanzada.

### K3s

Es la versión minimizada de kubernetes (K8s es el tamaño usado en Google Cloud o AWS), que gestiona de forma automática que software funciona en cada sensor o servidor. Y si un nodo falla, mueve su tarea al nodo más cercano. A diferencia de Kubernetes, es solo un fichero binario (sin dependencias) menor de 100 MB y por defecto está en SQLite3.

### Zero Trust Edge (ZTE)

Es una arquitectura de seguridad basada en la nube que combina redes (SD-WAN) y ciberseguridad (ZTNA) para verificar continuamente cada conexión y dispositivo en el "borde" de la red (donde se generan los datos), sin confiar en nadie por defecto. Es decir, reemplaza las VPNs tardicionales por una autenticación estricta (Zero Trust). Y su funcionamiento es, verificar la id y el estado del dispositivo del usuario para dar acceso a los datos necesarios para su función, y así obtiene una seguridad distribuida.

## WAsm

Es un código de bajo nivel en formato binario (bytecode) con el objetivo de tener ejecuciones mínimas y similares a C++ y Rust en un **sandbox**. En otras palabras, es "como un un Docker pero para funciones (no como entorno) y obligatoriamente ser ultraligero y rápido". Por ejemplo, en caso de BMW (empresa) se usa para modernizar el código legacy, el código que sea modular y que la gestión de datos sea ultrarápida.

## Cloud != Edge
-> Ubicación de datos
-> Latencia
-> Ancho de banda
-> Seguridad y privacidad
-> Dependencia de internet

## Casos en el futuro
## Caso de telecomunicaciones e infraestructura (???)

## Referencias y Bibliografías
https://www.cisco.com/site/us/en/learn/topics/computing/what-is-edge-computing.html
https://es.wikipedia.org/wiki/Computaci%C3%B3n_frontera
https://www.datacamp.com/es/blog/edge-vs-cloud-computing
https://eagerminds.medium.com/tiny-ml-enabling-smart-devices-to-make-intelligent-decisions-c8c8733307ba
https://blog.elhacker.net/2023/08/tesla-hardware-4-cuenta-con-la-mitad-de-ram-y-almacenamiento.html
https://www.ionos.es/digitalguide/servidores/know-how/k3s-vs-k8s/
https://www.fortinet.com/lat/resources/cyberglossary/zero-trust-edge


