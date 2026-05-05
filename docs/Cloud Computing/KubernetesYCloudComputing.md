# Kubernetes y el Cloud

Por _Sergio Pascual Morcillo_

## ¿Qué es Kubernetes?
__Kubernetes__ (K8s) es un orquestador de contenedores de código abierto y extensible. Fue diseñado para facilitar la automatización de tareas relacionadas con la gestión de infraestructuras multicontenedor empleando ficheros de configuración declarativos. Algunas de estas tareas son:
- El despliegue de aplicaciones.
- El escalado automático.
- El balanceo de carga entre contenedores.
- El lanzamiento de nuevas versiones de manera controlada.

Además, Kubernetes es _Cloud Native_. Esto significa que está diseñado desde cero para aprovechar al máximo las arquitecturas modernas: aplicaciones divididas en microservicios, entornos elásticos que crecen o se encogen según la demanda, y una automatización total frente a los fallos. Todo esto, independientemente de si se ejecuta en una nube pública (como AWS o Google Cloud) o en servidores locales.

## El clúster de Kubernetes
Kubernetes se ejecuta sobre un clúster (un conjunto de ordenadores), al cual se le conoce como __clúster de Kubernetes__. El clúster está dividido en dos secciones lógicas: el plano de control (___Control Plane___) y los nodos trabajadores (___Worker Nodes___). Cada componente del clúster ejecuta una serie de procesos para poder comunicarse con el resto de componentes.

<figure>
  <img src="https://kubernetes.io/images/docs/kubernetes-cluster-architecture.svg" alt="Diagrama de la arquitectura de un clúster de Kubernetes">
  <figcaption>Componentes de un clúster de Kubernetes</figcaption>
</figure>

El ___Control Plane___ es el cerebro del clúster. Se encarga de la toma de decisiones, así como de la detección y reacción ante eventos en este. El _Control Plane_ se puede ejecutar en cualquier máquina del clúster, la cual, habitualmente, no ejecuta contenedores. Puede estar formado por una o más máquinas para tener _alta disponibilidad_ (como sucede en entornos de producción reales, donde varios nodos, habitualmente un número impar de estos para realizar votaciones, coordinan las acciones del clúster). El _Control Plane_ ejecuta los siguientes procesos para cumplir con sus funciones:

- `kube-apiserver`. Es un servidor que se encarga de exponer una API Rest que permite al usuario realizar acciones en el clúster.
- `etcd`. Es una base de datos clave-valor distribuida que se utiliza para almacenar los datos relacionados con el estado del clúster.
- `kube-scheduler`. Es el proceso encargado de lanzar los contenedores en los diferentes _Worker Nodes_ según los recursos disponibles, políticas, restricciones, etc.
- `kube-controller-manager`. Es un proceso que ejecuta procesos que controlan el estado del clúster (_controllers_). Los _controllers_ se encargan de monitorizar el estado del clúster y realizar acciones para alcanzar lo que se conoce como el __estado deseado__ (hablaremos más adelante sobre él). Hay varios tipos de _controllers_:
  - _Node controllers_ que controlan el estado de un nodo.
  - _Job controllers_ que controlan el estado de una tarea.
  - _EndpointSlice controllers_ que controlan la conexión entre un _Service_ y los contenedores (hablaremos más tarde).
  - _ServiceAccount controllers_ que crean _ServiceAccounts_ en nuevos _namespaces_ (hablaremos más tarde).
- `cloud-controller-manager`. Es opcional, solo se ejecuta en entornos Cloud públicos como AWS o Google Cloud. Integra un clúster con los servicios de estos proveedores de Cloud para separar la gestión del clúster de la gestión del entorno Cloud.

Los ___Worker Nodes___, como su nombre indican, son los nodos del clúster encargados del trabajo sucio: ejecutar los contenedores. Cada _Worker Node_ ejecuta los siguientes procesos para recibir ordenes del plano de control:
- `kubelet`. Es el proceso que se encarga de recibir las ordenes del plano de control y ejecutarlas en un motor de contenedores, verificando constantemente su estado.
- `kube-proxy`. Es un proxy de red. Permite la comunicación entre contenedores sin que sea visible desde el exterior y es el punto de acceso a un conjunto de estos.
- Entorno de ejecución de contenedores. Es el proceso que ofrece la abstracción para ejecutar y gestionar los contenedores. Algunos ejemplos son __Docker__, __containerd__ o __CRI-O__.

Además, existen los ___Addons___ que son complementos que extienden las características del clúster aportando nuevas funcionalidades. Podemos ver los _addons_ existentes en [esta página](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

> La herramienta oficial para crear un clúster de Kubernetes es `kubeadm` aunque existen otras alternativas como `minikube`, `kind`, `k3s` o `microk8s` que realizan la instalación básica ellos mismos y añaden algunos _addons_ para mejorar la experiencia de usuario.

## Los recursos básicos de Kubernetes
Kubernetes utiliza una serie de recursos para montar las infraestructuras de contenedores. Estos recursos se configuran declarativamente en manifiestos __YAML__, los cuales la API interpreta para contruir la infraestructura. 

### Pod
Es la __unidad básica__ que se puede crear y gestionar en Kubernetes. Un __pod__ envuelve un conjunto de contenedores (uno o más) y sus recursos asociados (volúmenes, interfaces de red o configuración) en una vaina que ofrece un contexto de ejecución compartido para estos, como si de una vaina de guisantes se tratase (de ahí su nombre en inglés).

Un pod presenta las siguientes características:
- __Se ejecuta en un único nodo__ que le ha sido asignado por el _Control Plane_. Un nodo puede ejecutar varios pods.
- __Los contenedores del pod comparten recursos__. Es decir, que los contenedores del pod se identifican con una única IP (clúster IP) y comparten los volúmenes. Los contenedores se pueden comunicar entre ellos mediante `localhost`.
- __Son efímeros y desechables por naturaleza__. Esto implica que los pods no guardan datos internos (pues tarde o temprano se eliminarán, incluso estando en un volumen del pod) y, ante cualquier problema, el pod se desechará sin contemplaciones. 

<figure>
  <img src="https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg" alt="Diagrama de pods presentes en un nodo">
  <figcaption>Pods ejecutandose en un nodo de Kubernetes</figcaption>
</figure>

> En entornos reales, no se crean pods a mano, se utlizan __ReplicaSets__.

### ReplicaSet
Un __ReplicaSet__ es un __controlador__ que asegura que haya un número de réplicas de un pod ejecutándose en todo momento. Cuando se crea un _ReplicaSet_, el usuario define un número de pods que quiere activos, lo que se conoce como un __estado deseado__.

> Realmente, un __estado deseado__ es lo que el usuario declara en los manifiestos YAML.

> Como pasa con los pods, en entornos reales no se crean ReplicaSets a mano, sino que se delegan en un __Deployment__.

### Deployment
Un __Deployment__ es un objeto de Kubernetes que indica la infraestructura de contenedores que se quiere desplegar y, además, mantiene el __estado deseado__ de esta (declarado en un manifiesto YAML).

Cuando se sube un Deployment al _Control Plane_, se generará en este un __Deployment Controller__ que será el encargado de mandar las ordenes a los _Worker Nodes_. El _controller_ generará una serie de __ReplicaSets__ que a su vez crearán los __Pods__ necesarios para cumplir con el __estado deseado__, los cuales serán asignados a los diferentes nodos del clúster por el `kube-scheduler`.

> Una ventaja de los Deployments de Kubernetes, es que nos permite realizar _rolling updates_ (cambios de versión graduales sin romper la disponibilidad) de las aplicaciones desplegadas en los contenedores, así como _rollbacks_ o _rollouts_ para volver a versiones anteriores.

Un ejemplo de manifiesto YAML para un Deployment es el siguiente:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: example-container
        image: example-image
        ports:
        - containerPort: 80
```

El manifiesto consta de 4 partes:
- `apiVersion`: la versión de la API de Kubernetes ejecutándose en el clúster.
- `kind`: el tipo de recurso que crea el manifiesto. En este caso, un Deployment.
- `metadata`: datos para identificar el recurso (nombre, etiquetas, _namespaces_).
- `spec`: la especificación del __estado deseado__ del Deployment. También tiene varias partes:
  - `replicas` y `selector`: indican al ReplicaSet el número de réplicas y cuáles debe controlar usando etiquetas.
  - `template`: define el pod. En la sección `metadata` se definen las etiquetas del pod y en la sección `spec` se definen los contenedores del pod (nombre, imagen, configuraciones como el mapeo de puertos, etc.).
  > Si queremos definir más de un tipo Pod en el Deployment, podemos crear otros ficheros con estas cuatro partes (que son Deployments en sí). Si queremos que estén en el mismo manifiesto, concatenar los manifiestos separándolos por `---`.

### Servicios
Debido a la naturaleza efímera de los Pods, se puede dar la siguiente problemática: cuando un pod se crea, se le asigna una IP automáticamente; si queremos comunicar dos Pods (por ejemplo, un _backend_ con una base de datos) ¿cómo podemos saber sus IPs?. Un __servicio__ viene a solventar esta problemática creando una abstracción a nivel de red que define un punto de acceso único y estable a un conjunto de Pods (marcados por una etiqueta).

Existen varios tipos de servicios, pero destacamos los siguientes:
- `ClusterIP`: es el servicio por defecto y sirve para comunicación interna entre Pods.
- `NodePort`: permite acceder a un servicio desde el exterior del clúster usando la IP de un nodo y un puerto.
- `LoadBalancer`: crea un balanceador de carga en un proveedor de infraestructura Cloud que reparte las peticiones entre los Pods. Es una versión más refinada de `NodePort` disponible solo si se usa un entorno Cloud.

<figure>
  <img src="https://kubernetes.io/docs/tutorials/kubernetes-basics/public/images/module_04_labels.svg" alt="Diagrama de un servicio en Kubernetes">
  <figcaption>Diagrama de un servicio en Kubernetes</figcaption>
</figure>

> Hemos visto que con `NodePort` podemos acceder a un servicio sabiendo la IP de un nodo, pero existe otra forma de exponer servicios sin conocer la infraestructura del clúster: mediante un __Ingress__. Un __Ingress__ nos permite exponer rutas HTTP/HTTPS desde el exterior del clúster que se enrutan a servicios ejecutandose dentro de él. El encargado de cumplir con las reglas de los Ingress es un __Igress controller__ que actúa como un proxy inverso. Actualmente, existe otra opción que permite un enrutamiento más avanzado: __Gateway API__.


### ConfigMaps y Secrets
Los __ConfigMaps__ y los __Secrets__ son dos tipos de objetos de la API de Kubernetes que sirven para almacenar valores de manera que estos no se _hardcodeen_ en la imagen del contenedor. Por una parte, los __ConfigMaps__ son ficheros que almacenan parejas clave-valor no confidenciales. Estas parejas clave-valor se introducen en los contenedores de un Pod como variables de entorno. Por otra parte, los __Secrets__ almacenan información sensible (como pueden tokens de acceso o contraseñas) que Kubernetes trata con más consideración. Los _Secrets_ se suelen almacenar codificados en Base64 en los manifiestos YAML.

> Los __ConfigMaps__ y los __Secrets__ se incluyen en el apartado `env:` de la configuración de un contenedor. Con `name:` se le da nombre a la variable de entorno y con `valueFrom` se determina si proviene de un ConfigMap (`configMapKeyRef:` seguido del nombre del ConfigMap y el nombre de la clave) o de un Secret (`secretKeyRef:` seguido de igual forma del nombre del Secret y el nombre de la clave)

### Volúmenes y almacenamiento
Como hemos mencionado en la sección del Pod, los datos internos que almacenan se pierden cuando el Pod se destruye. Además, es posible que dos instancias de un contenedor pretendan interactuar con los mismos datos. Para solucionar estos problemas de persistencia de datos y uso compartido, Kubernetes nos ofrece los __volúmenes__.

Hay varios tipos de __volúmenes__ pero nos centraremos en el que resuelve las casuísticas que hemos descrito anteriormente: los __Persistent Volumes (PV)__. Un PV es un objeto de la API de Kubernetes que abstrae un trozo de almacenamiento real que se provee al clúster y que tiene una duración independiente de los Pods que lo usan. Al ser una abstracción de un sistema de almacenamiento, se pueden usar varias implementaciones como: una carpeta compartida por NFS, un disco duro en red mediante iSCSI o incluso una carpeta del nodo (si solo tenemos un nodo en el clúster), entre otros.

Con el almacenamiento configurado en el clúster mediante los PV, para que un Pod pueda usarlos debe utilizar un __Persistent Volume Claim (PVC)__. Un PVC en una petición de uso de un PV por parte de un Pod en la que se define el espacio necesario y el tipo de acceso al PV. Cuando se programa un Pod con una PVC, Kubernetes realiza el _bind_ con un PV que cumple con los criterios de la PVC, y de esta forma, el Pod puede almacenar datos en el PV.

Como hemos mencionado en el párrafo anterior, una PVC define el tipo de acceso al PV. A continuación, exponemos los tipos de acceso que existen:
- `ReadWriteOnce`: un único nodo puede montar el volumen y realizar operaciones de lectura y escritura. Los Pods ejecutandose en el mismo nodo pueden acceder a dicho volumen.
- `ReadWriteOncePod`: un único Pod puede montar el volumen y realizar operaciones de lectura y escritura.
- `ReadOnlyMany`: varios nodos pueden montar el volumen exclusivamente para leer datos.
- `ReadWriteMany`: varios nodos pueden montar el volumen y realizar operaciones de lectura y escritura.

El siguiente código es un ejemplo de un manifiesto YAML de un Deployment que utiliza un PVC para usar un volumen:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: my-pvc  # Name of the Persistent Volume Claim
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi  # Request 1 Gigabyte of storage
```

> En entornos reales (especialmente Cloud), no se crean PVs, si no que se crea una __StorageClass__ que permite el aprovisionamiento dinámico, es decir, Kubernetes se encarga de crear el disco, asignarlo a un PV y realizar el _bind_ con una PVC.

### Namespaces y control de recursos
Un __namespace__ es un mecanismo para aislar grupos de recursos dentro de un clúster, o en otras palabras, crear clústeres virtuales. Los nombres de los objetos que creamos en un _namespace_ son únicos, pero se pueden repetir en otros _namespaces_. Como mínimo, existen cuatro _namespaces_ en un clúster:
- `default` que, como su nombre indica, es el que Kubernetes usa por defecto.
- `kube-node-lease` que gestiona información de salud sobre los nodos del clúster.
- `kube-public` que expone datos públicos.
- `kube-system` que agrupa los objetos internos de Kubernetes.

En cada _namespace_ podemos configurar los recursos (como uso de CPU o memoria) que se pueden utilizar mediante __ResourceQuotas__. También se puede limitar la comunicación entre Pods o con el exterior de un mismo _namespace_ usando __NetworkPolicies__ las cuales actuan como un firewall.

> Kubernetes también permite configurar los recursos máximos que puede consumir un contenedor. Esto puede ser útil para seguir estrategias de escalado vertical (aumentar los recursos asignados) o horizontal (lanzar más instancias de los contenedores).

## Entornos Cloud públicos
Como hemos mencionado durante el transcurso de esta entrada, Kubernetes se ejecuta en un clúster de computadores. Esto significa que la capacidad de computo del clúster está limitada por sus características de _hardware_ (procesador, memoria, conectividad, almacenamiento), las cuales, si son muy altas, pueden implicar un gran gasto de adquisición y mantenimiento, sin contar los costes de administración. 

Los entornos Cloud públicos como __Amazon Web Services (AWS)__, __Google Cloud__ o __Microsoft Azure__ nos permiten hospedar un clúster en sus diferentes centros de datos, además de proveer servicios para facilitar la gestión de componentes en el clúster o enriquecer las funcionalidades de este (por ejemplo, crear `LoadBalancers` o usar __StorageClasses__ para crear discos _on the fly_).

### AWS
AWS fue el pionero de los entornos Cloud públicos y es el más usado actualmente. Como el resto de proveedores de infraestructura Cloud, AWS permite crear clústers de Kubernetes usando sus recursos. Se distinguen dos tipos de enfoques a la hora de crear un clúster:
- El enfoque autogestionado, en el que el propio usuario crea las máquinas del clúster (intancias EC2) y las configura y gestiona manualmente.
- El enfoque gestionado mediante __EKS (Elastic Kubernetes Service)__ en el cual __AWS__ se encarga de la gestión del ___Control Plane___ desplegando varias instancias en diferentes __AZs (zonas de disponibilidad)__ para garantizar alta disponibilidad y el __usuario__ gestiona los ___Worker Nodes___ en diferentes AZs de igual forma.

## Taller
En este [repositorio](https://github.com/spm73/kubernetes-workshop) podeis encontrar un taller para realizar el despliegue de un aula inteligente basada en Home Assistant con un módulo de visión artificial para poder poner en práctica los conceptos básicos de Kubernetes. En el repositorio podreis acceder a unas [diapositivas](https://view.genially.com/69de8ca748ebb6b0cef431de) que sintetizan los conceptos más importantes de Kubernetes y muestran unos ejemplos de manifiestos YAML.

## Referencias
[Learn Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

[Kubernetes concepts](https://kubernetes.io/docs/concepts/)

[De cero a cien con Docker y Kubernetes](https://www.palentino.es/pdfs/De-cero-a-cien-con-Docker-y-Kubernetes.pdf)

[Kubernetes Deployment YAML File with Examples](https://spacelift.io/blog/kubernetes-deployment-yaml)
