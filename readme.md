# 🚀 Infraestructura Web Contenedorizada con Docker Compose

## 1. Introducción
Este repositorio documenta el diseño y despliegue de un entorno de servidor local sobre **Ubuntu**, utilizando una arquitectura de microservicios. El objetivo es eliminar la dependencia del software instalado directamente en el sistema operativo, permitiendo que todo el stack tecnológico sea portable, reproducible y fácil de mantener en diferentes entornos de desarrollo o producción.

### 🐳 El Concepto de Docker: La Analogía del Contenedor
Para entender Docker, podemos usar el ejemplo de la industria del transporte marítimo. Antiguamente, cargar un barco era caótico porque cada mercancía tenía formas y pesos distintos. La invención del **contenedor estandarizado** revolucionó esto: no importa si dentro hay muebles, electrónica o comida; el puerto solo ve una caja estándar que encaja perfectamente en cualquier barco o camión.

En el software, Docker hace lo mismo. En lugar de enviar código y esperar que el servidor tenga la versión exacta de PHP o MySQL, enviamos un "contenedor" que incluye el código, las librerías y la configuración necesaria. Esto garantiza que la aplicación funcione siempre, sin importar si el sistema anfitrión es Linux, Windows o macOS, eliminando el famoso conflicto de *"en mi máquina sí funciona"*.



---

## 2. Arquitectura del Stack
Hemos orquestado un ecosistema de 5 servicios independientes. Cada uno cumple una función específica y se comunica con los demás a través de una red virtual privada, garantizando seguridad y eficiencia.

| Contenedor | Imagen / Servicio | Función Detallada |
| :--- | :--- | :--- |
| **`myappnginx`** | Nginx | Actúa como el punto de entrada (puerto `89`), gestionando el tráfico HTTP y sirviendo archivos estáticos con alta velocidad. |
| **`miAppPHP`** | PHP 8.1-FPM | Se encarga de procesar toda la lógica del lado del servidor y la comunicación con la base de datos mediante el protocolo FastCGI. |
| **`MiAppMySQL`** | MySQL 8.0 | Almacena toda la información relacional de forma estructurada, gestionando la integridad de los datos de los usuarios registrados. |
| **`miappPhpMyAdmin`** | phpMyAdmin | Proporciona una interfaz web intuitiva en el puerto `8089` para realizar tareas de administración de bases de datos sin usar la consola. |
| **`portainer`** | Portainer CE | Permite monitorizar el estado de los contenedores, revisar logs y gestionar volúmenes de forma visual a través del puerto `9443`. |

---

## 3. Fundamentos y Preguntas Técnicas

### 💡 Conceptos Esenciales de Contenerización
Para dominar este entorno, hemos abordado diversas cuestiones teóricas que diferencian a Docker de otras tecnologías:

* **¿Qué es una Imagen vs. un Contenedor?** Una **imagen** es una plantilla estática de "solo lectura" que contiene el sistema de archivos y las dependencias (como una fotografía del sistema). El **contenedor** es la instancia de ejecución creada a partir de esa imagen; es un proceso vivo que consume recursos y realiza tareas en tiempo real.
* **¿Docker o LXC?** Aunque ambos son contenedores Linux, los **LXC** virtualizan sistemas operativos completos. Docker, en cambio, está orientado a aplicaciones, utilizando un sistema de capas que permite compartir el núcleo del sistema de forma mucho más ligera y eficiente.
* **Persistencia de Datos:** Por naturaleza, los contenedores son efímeros; si se borran, sus datos internos mueren con ellos. Para evitar esto, hemos configurado **volúmenes**, que permiten que los datos de **`MiAppMySQL`** residan en el disco duro del host, permaneciendo intactos incluso si el contenedor se recrea.
* **Versatilidad de Despliegue:** Docker permite desplegar prácticamente cualquier servicio, desde aplicaciones web (como esta) hasta herramientas de análisis de datos, servidores de correo o nodos de blockchain, siempre que el software pueda ejecutarse sobre un núcleo Linux.



---

## 4. Especificaciones de Configuración

### ⚙️ Orquestación (Docker Compose)
El archivo `docker-compose.yml` funciona como el plano arquitectónico del proyecto. Define no solo qué imágenes descargar, sino cómo deben relacionarse entre sí a través de la red **`appnet`**. Esto permite que los contenedores utilicen DNS internos: por ejemplo, PHP no necesita saber la IP de la base de datos, solo necesita llamar a `MiAppMySQL`.

### 🌐 Configuración de Nginx
La configuración de Nginx en este entorno difiere de una instalación tradicional "Standalone". En lugar de buscar un proceso local, Nginx actúa como un proxy inverso que redirige las peticiones de scripts a la dirección `miAppPHP:9000`. Además, las rutas de directorios configuradas deben coincidir exactamente con los puntos de montaje de los volúmenes internos para que el servidor encuentre el código fuente.

---

## 5. Resolución de Incidencias (Logbook)

Durante la fase de implementación, se documentaron y resolvieron los siguientes retos técnicos:

### ❌ Gestión de Rutas y Volúmenes
**Problema:** El contenedor **`myappnginx`** fallaba al arrancar porque Docker intentaba montar el archivo `default.conf` como si fuera una carpeta, debido a que el archivo original no estaba presente en el host durante el primer inicio.
**Solución:** Se realizó una limpieza profunda de los directorios creados por error y se verificó la existencia del archivo físico antes de levantar el stack nuevamente.

### ❌ Evolución de Sintaxis (Docker V1 vs V2)
**Problema:** Al ejecutar comandos con el guion clásico (`docker-compose`), el sistema Ubuntu devolvía un error de "comando no encontrado".
**Solución:** Se actualizó el flujo de trabajo al estándar moderno **Docker V2**, utilizando la sintaxis nativa `docker compose` que viene integrada como un plugin del motor principal.

### ❌ Sensibilidad a Mayúsculas y Minúsculas
**Problema:** La aplicación PHP lanzaba una excepción indicando que la tabla `landing.Usuarios` no existía, a pesar de estar creada en la base de datos.
**Solución:** Al trabajar sobre Linux (dentro del contenedor), el motor de base de datos es *case-sensitive*. Se estandarizó todo el código para que los nombres de las tablas coincidieran exactamente con los de la base de datos.

### ❌ Error de Cabeceras HTTP (Headers already sent)
**Problema:** Aparecían advertencias de PHP que impedían la redirección automática tras un registro, debido a que se estaba enviando texto al navegador antes de la función `header()`.
**Solución:** Se reorganizó la estructura del archivo `registro.php`, moviendo toda la lógica de redirección y el manejo de búfer al inicio del script, antes de cualquier etiqueta HTML o comando `echo`.

### 📧 Autenticación SMTP
**Problema:** El servicio de envío de correos fallaba por restricciones de seguridad de Google al intentar conectar desde un entorno no verificado.
**Solución:** Se implementaron "Contraseñas de Aplicación" específicas, permitiendo que PHPMailer realizara la validación de usuarios de forma segura y sin bloqueos de autenticación.
