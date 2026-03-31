**Autor:** Alberto Boronat López  
**Fecha:** Marzo 2026

---

## 1. Introducción

En el contexto actual, donde los ciberataques, filtraciones de contraseñas, de datos y la seguridad del usuario promedio están en constante compromiso —así como el creciente uso de la nube como almacenamiento de datos—, aprender a montar nuestros propios servicios puede ser una gran herramienta tanto de aprendizaje como para usarla a futuro. 

Nosotros, los usuarios, cada vez estamos perdiendo más nuestra **soberanía digital**, así que crear un **"Homelab"** o servidor de toda la vida nos puede dar el control absoluto de nuestros recursos:

* Copias de seguridad.
* Gestores de contraseñas.
* Bloqueadores de red.
* VPNs.

> [!CAUTION]
> **Responsabilidad:** Aunque esto ofrece libertad, te hace única y exclusivamente responsable de su seguridad y disponibilidad. Si pierdes el acceso o sufres una intrusión por una mala configuración, lo pierdes todo.

---

## 2. ¿Cómo se podría montar?

Para montar esto no necesitamos ningún superordenador, racks de servidor gigantes ni nada parecido. Podemos apañarnos con algún **portátil viejo**, ya que casi cualquier procesador de hace 10 años nos permitirá realizar la gran mayoría de tareas. 

* Mantenimiento preventivo: Se recomienda retirar la batería y operar con corriente directa debido a que las baterías de litio, si están enchufadas mucho tiempo, se acaban inflando.


##  3. Configuración a Nivel de Kernel (ACPI y Mitigación Térmica)

Lo mejor para montar esto es Ubuntu Server, ya que es código abierto.

### 3.1 Mini-Tutorial: Instalación de Ubuntu Server

    *A diferencia de la versión de escritorio, Ubuntu Server carece de interfaz gráfica (GUI), lo que libera todos los recursos de la CPU y la RAM para nuestros servicios de red.*

* Descargar el ISO desde la [pagina web](https://ubuntu.com/download/server) de Ubuntu y utilizar Rufus (si estamos en Windows) para flashear el ISO en el PEN. Después insertaremos el USB y desde la BIOS pondremos el orden de arranque para que inicie desde el USB.

### 3.2 Asistente de instalación

* Además de lo típico de elegir idioma, etc., tenemos que configurar la red. **Es muy importante que le asignemos una IP estática como por ejemplo 192.168.1.100** para que los puertos que abramos siempre apunten a la máquina correcta.

* Configurar el nombre del servidor y contraseña 

* **Paso crítico**: En el asistente, cuando pregunte por software adicional, debemos marcar la instalación de **OpenSSH Server**, ya que este paquete nos va a permitir conectarnos al server desde otro dispositivo.

### 3.3 Systemd y cierre de tapa

Por diseño, Linux está diseñado para que siempre que se cierre la tapa del portátil perdamos la conexión de red, así que debemos asegurarnos de modificar systemd para que esto no pase y tengamos el servidor siempre operativo.


#### 1.Editamos el archivo de configuración base:
<pre>
Bash
sudo nano /etc/systemd/logind.conf
</pre>

#### 2. Modificamos la directiva del interruptor de la tapa:

<pre>
Ini, TOML
HandleLidSwitch=ignore
</pre>

Al cambiar el valor por defecto (suspend) a ignore, le indicamos explícitamente al servicio de inicialización del núcleo que descarte las interrupciones de hardware generadas por el sensor magnético (efecto Hall) de la placa base al cerrar la tapa.

#### 3. Reiniciamos para aplicar cambios.

<pre>
Bash
sudo systemctl restart systemd-logind
</pre>


### 3.4 Apagado de la pantalla

Aunque el sistema ya no se suspenda, la pantalla sigue recibiendo corriente y eso, a la larga, puede acortar la vida útil del portátil. Para evitar esto, añadiremos una instrucción al planificador de tareas **(cron)** que se inyecte directamente en el sistema de archivos virtual **(sysfs)** del hardware en cada arranque.


#### 1. Abrir el editor de tareas programadas del superusuario

<pre>
Bash 
sudo crontab -e
</pre>

#### 2. Añadir la siguiente línea al final del archivo:

<pre>
Bash
@reboot echo 0 | tee /sys/class/backlight/intel_backlight/brightness
</pre>

La directiva @reboot asegura que se ejecute al encender la máquina. El comando toma un valor binario nulo (0) y, usando tee, lo escribe con privilegios directamente en la interfaz del controlador gráfico de la placa base (backlight/brightness). Esto corta físicamente la energía de la pantalla.

## 4. Software

La regla de oro para instalar software en estos entornos es no instalarlo directamente en el sistema base, sino utilizar **dockers** para evitar conflictos de dependencias.

### 4.1 Docker y Portainer

El primer paso es instalar el motor de Docker para poder aislar cada aplicación y sus dependencias. Para más comodidad, vamos a utilizar Portainer, que es una interfaz gráfica web que nos permite desplegar, actualizar y monitorizar todos nuestros dockers.

### 4.2 Enrutamientos: Nginx proxy manager

Si queremos alojar varios servicios, no podemos abrir puertos diferentes de nuestro router para cada uno de ellos. Lo más eficiente es utilizar un **proxy inverso**, que es básicamente un servidor intermedio que actúa como muro delante de los servidores y aplicaciones web, interceptando todas las peticiones, ocultando nuestra IP real y permitiéndonos balancear la carga de tráfico y los certificados de las webs. Por lo tanto, este proxy es el único que está expuesto a internet, escuchando los puertos estándar como el 80 y el 443. Cuando recibe una petición, redirige el tráfico hacia el puerto específico del servicio.

Para conseguir esto vamos a utilizar Nginx Proxy Manager, que es un servicio que cuenta con interfaz y nos permite cifrar todas las conexiones.


### 4.3 Servicios que podemos integrar

Ahora que tenemos las bases para poder desplegar el servidor de manera eficiente, toca ver qué servicios podemos integrar. Todo depende de nuestras necesidades. Voy a poner dos que son bastante populares y nos pueden dar alternativas a Google u otras apps.

* Nextcloud: Es la alternativa más famosa a Google Drive. Nos permite sincronizar fotos, vídeos, archivos, calendarios, documentos de texto colaborativos, etc.

* Vaultwarden: Compatible con Bitwarden, nos permite almacenar nuestras propias contraseñas.


## 5. Seguridad de la red

### 5.1 Adblocker

Para montar nuestro propio adblocker vamos a implementar un sinkhole DNS utilizando **Pi-hole**. Lo que hace esto es básicamente interceptar las peticiones a nivel de red (puerto 53), buscando en una lista negra de servidores. Si está, este intercepta la petición y le devuelve una IP falsa o una ruta nula como 0.0.0.0, evitando así publicidad, malware, etc.


<pre>
Bash
curl -sSL https://install.pi-hole.net | bash
pihole -a -p  # Establece o cambia la contraseña web tras la instalación
</pre>

### 5.2 VPN 

Para montar la VPN en nuestra red instalaremos WireGuard. Al ser un protocolo integrado en el núcleo de Linux, ofrece la máxima velocidad en hardware antiguo. Utilizaremos el instalador automatizado PiVPN, que detecta automáticamente a Pi-hole y configura el enrutamiento para que el bloqueo de anuncios funcione a través del túnel VPN.

<pre>
Bash
curl -L https://install.pivpn.io | bash
</pre>


### 5.3 VPN para ordenadores externos

Si queremos conectar nuestro ordenador remotamente al servidor, tenemos que hacer lo siguiente:

* 1. Crear un nuevo perfil 

<pre>
Bash
pivpn add
</pre>

* 2. El asistente te pedirá un nombre (por ejemplo, portatil-trabajo). Al finalizar, PiVPN generará un archivo de configuración .conf.

* 3. Pasarnos este archivo al ordenador 

* 4. Instalar la aplicación cliente de WireGuard en tu Windows, macOS o Linux, hacer clic en "Importar túnel desde archivo" y seleccionar el documento que acabas de descargar. Cuando actives la conexión, todo el tráfico de tu ordenador viajará encriptado hasta tu casa.

### 5.4 VPN en nuestro móvil


* 1. Crear un nuevo perfil 

<pre>
Bash
pivpn add
</pre>

* 2. Generar un código QR para nuestro móvil

<pre>
Bash
pivpn qr
</pre>

* 3. Abre la app de WireGuard en tu teléfono, selecciona el botón "+" y elige "Escanear desde código QR". Apunta con la cámara a la pantalla de tu ordenador y listo. 

## 6. Mantenimiento Predictivo y Teoría de Fiabilidad

Para no tener que esperarnos a que algo se rompa, podemos adelantarnos usando diferentes plataformas de código libre como Netdata, Zabbix o la dupla Prometheus + Grafana.

Estas herramientas miden el desgaste de la CPU, discos duros, temperaturas, cargas, etc., lo que nos permite medir 2 cosas fundamentales:

MTBF (Mean Time Between Failures / Tiempo medio entre fallos): Es el tiempo promedio que el servidor opera correctamente entre dos fallos. Se calcula dividiendo el tiempo total de operación entre el número de caídas.

MTTR (Mean Time To Repair / Tiempo medio de reparación): Es el tiempo promedio que se tarda en recuperar el sistema y volver a dar servicio tras una caída.

Esto nos permite mediante la siguiente ecuación:

$$A = \frac{MTBF}{MTBF + MTTR}$$


¿Como podemos aprovecharnos de estos datos? 


* 1. Aumentar el MTBF: Si usamos Zabbix para monitorizar el exceso de temperatura de nuestro viejo portátil y cambiamos la pasta térmica a tiempo.

* 2. Disminuir el MTTR: Como hemos configurado todo en dockers, si nuestro portátil muere no tenemos que reinstalar y reconfigurar todo desde cero. Simplemente copiamos las carpetas de Docker a otro ordenador, ejecutamos el servicio y la red vuelve a estar lista.

## 7. Bibliografía

*  Sostenibilidad y Reutilización de Hardware:
Chhillar, K., Tomar, D., & Kanchan, D. K. (2025). Hardware Reuse and Recycling in Sustainable Computing: Extending Device Lifetimes Through Modular Design. International Journal of Scientific Research in Engineering and Management (IJSREM).

*  Criptografía y Rendimiento de Red (VPN):
Donenfeld, J. A. (2017). WireGuard: Next Generation Kernel Network Tunnel. NDSS Symposium.

*  Mantenimiento y Teoría de Fiabilidad:
Conceptos matemáticos de confiabilidad, aplicados a las métricas de Mean Time Between Failures (MTBF) y Mean Time To Repair (MTTR), fundamentales para las metodologías modernas de operaciones y Site Reliability Engineering (SRE).

*  Seguridad Perimetral (DNS Sinkholing):
SANS Institute. SEC595: Applied Data Science and AI/Machine Learning for Cybersecurity Professionals.

*  Soberanía Digital (Self-Hosting):
Comisión Europea (2025). The Commission moves forward on cloud sovereignty. 