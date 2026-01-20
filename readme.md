# 🚀 Infraestructura Web Contenedorizada con Docker Compose

## 1. Introducción
Este repositorio documenta el proceso completo de configuración de un servidor local sobre **Ubuntu** mediante el uso de microservicios. A diferencia de una instalación tradicional, este enfoque permite aislar cada componente del sistema, garantizando que el entorno de desarrollo sea idéntico al de producción.

### 🐳 ¿Qué es Docker?
Docker es una plataforma que permite empaquetar una aplicación y todas sus dependencias en un **contenedor** ligero. Para entenderlo de forma sencilla: es como un contenedor de carga marítimo. No importa qué haya dentro (PHP, MySQL o Nginx), el contenedor tiene un tamaño estándar que encaja perfectamente en cualquier "barco" (ordenador o servidor), evitando el clásico problema de *"en mi máquina sí funciona"*.

---

## 2. Arquitectura del Stack
El proyecto se basa en un ecosistema de 5 servicios interconectados que gestionan la lógica, los datos y la administración visual:

| Contenedor | Servicio | Función Principal |
| :--- | :--- | :--- |
| **`myappnginx`** | Nginx | Servidor web que recibe las peticiones en el puerto `89`. |
| **`miAppPHP`** | PHP 8.1-FPM | Procesador de scripts encargado de la lógica de registro. |
| **`MiAppMySQL`** | MySQL 8.0 | Motor de base de datos para el almacenamiento de usuarios. |
| **`miappPhpMyAdmin`** | phpMyAdmin | Interfaz web para la gestión de la base de datos (Puerto `8089`). |
| **`portainer`** | Portainer CE | Panel de control visual para la gestión de todo el entorno Docker. |

---

## 3. Especificaciones de Configuración

### ⚙️ Orquestación (Docker Compose)
El archivo `docker-compose.yml` actúa como el director de orquesta. Automatiza la creación de la red interna **`appnet`**, permitiendo que los servicios se comuniquen por nombre. 

* **Persistencia de datos:** Se implementó el volumen `./mysql_data:/var/lib/mysql`. Esto asegura que la información de la base de datos no se destruya si el contenedor se detiene o se borra.
* **Aislamiento:** Cada componente opera en su propia parcela de memoria, mejorando la seguridad y estabilidad del sistema.

### 🌐 Nginx: Docker vs. Standalone
Configurar Nginx en Docker presenta diferencias críticas frente a una instalación tradicional:
1.  **Directivas FastCGI:** En lugar de usar `127.0.0.1`, Nginx se comunica con PHP usando el nombre del servicio: `fastcgi_pass miAppPHP:9000;`.
2.  **Rutas de Archivos:** Las rutas configuradas en el servidor (`/var/www/landing/`) corresponden a la estructura interna del contenedor, no a la ruta física de la máquina host.

---

## 4. Resolución de Incidencias Técnicas

Durante el desarrollo se gestionaron y solucionaron diversos retos técnicos comunes en entornos de contenedores:

### ❌ Gestión de Rutas y Volúmenes
**Problema:** Error de montaje donde Nginx fallaba al intentar leer un archivo de configuración (`default.conf`) porque Docker lo interpretaba como un directorio.
**Solución:** Se corrigió la ruta absoluta en el archivo YAML y se aseguró la existencia del archivo físico en el host para evitar que Docker creara carpetas vacías por defecto.

### ❌ Evolución de Comandos (V1 a V2)
**Problema:** El sistema no reconocía el comando `docker-compose` (con guion).
**Solución:** Migración al estándar moderno **Docker V2**, utilizando la sintaxis nativa `docker compose` integrada directamente en el motor de Docker.

### ❌ Sensibilidad a Mayúsculas (Case Sensitivity)
**Problema:** Error `mysqli_sql_exception: Table 'landing.Usuarios' doesn't exist`.
**Solución:** Dado que los contenedores corren sobre Linux, los nombres de las tablas distinguen entre mayúsculas y minúsculas. Se estandarizó la nomenclatura en el código PHP para coincidir exactamente con la base de datos.

### ❌ Conflictos de Cabeceras HTTP
**Problema:** Error `Warning: Cannot modify header information`.
**Solución:** Reestructuración del archivo `registro.php` para garantizar que no existiera ninguna salida de texto (`echo`) antes de las instrucciones de redirección `header()`.

### 📧 Autenticación de Correo (SMTP)
**Problema:** Bloqueos de seguridad al intentar enviar correos de validación.
**Solución:** Implementación de "Contraseñas de aplicación" de Google, permitiendo que **`miAppPHP`** se autentique de forma segura sin comprometer la cuenta principal.
