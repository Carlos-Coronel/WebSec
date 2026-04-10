# 🛡️ Laboratorio de Ciberseguridad Web (WebSec Lab)

Este proyecto proporciona un entorno de laboratorio **seguro, aislado y reproducible** para la práctica de auditoría web y vulnerabilidades del **OWASP Top 10**. Utiliza Docker Compose para desplegar aplicaciones deliberadamente vulnerables en un entorno de red privada.

## 🚀 Inicio Rápido (Quick Start)

Para desplegar todo el laboratorio en menos de un minuto:

1.  Asegúrate de tener **Docker Desktop** instalado y en ejecución.
2.  Abre una terminal (PowerShell o CMD) en la carpeta raíz de este proyecto.
3.  Ejecuta el siguiente comando:
    ```bash
    docker-compose up -d
    ```

---

## 🏗️ Arquitectura del Laboratorio

El laboratorio está diseñado siguiendo buenas prácticas de aislamiento y persistencia:

| Servicio | Puerto Host | Puerto Contenedor | Descripción | Credenciales |
| :--- | :--- | :--- | :--- | :--- |
| **DVWA** | `8001` | `80` | PHP/MySQL - Entorno clásico de pruebas para ataques tradicionales. | `admin` / `password` |
| **OWASP Juice Shop** | `8002` | `3000` | Node.js/Angular - Aplicación moderna SPA con retos interactivos. | (Explorar en el lab) |
| **WebGoat** | `8003` | `8080` | Java/Spring Boot - Plataforma de lecciones guiadas sobre OWASP. | (Registro propio) |
| **WebWolf** | `8004` | `9090` | Java - Herramienta complementaria para simular ataques externos. | (Mismas de WebGoat) |

### 🔍 ¿Qué es cada aplicación exactamente?

Este laboratorio combina tres de las herramientas más respetadas en la enseñanza de la ciberseguridad:

1.  **DVWA (Damn Vulnerable Web Application)**:
    *   **Propósito**: Es una aplicación web en PHP/MySQL diseñada para ser increíblemente vulnerable. Es ideal para practicar ataques clásicos como SQL Injection, XSS y Command Injection de forma directa.
    *   **Lo mejor**: Su sistema de niveles de seguridad (`Low`, `Medium`, `High`, `Impossible`) te permite ver cómo los programadores intentan (y a veces fallan) corregir las vulnerabilidades.
2.  **OWASP Juice Shop**:
    *   **Propósito**: Representa una tienda moderna de jugos escrita en Node.js, Express y Angular. A diferencia de DVWA, esta es una **SPA (Single Page Application)** moderna, lo que la hace perfecta para practicar ataques contra APIs, filtración de datos sensibles y lógicas de negocio.
    *   **Lo mejor**: Incluye un sistema de retos gamificado (Score Board) que se desbloquea conforme encuentras vulnerabilidades ocultas.
3.  **WebGoat & WebWolf**:
    *   **Propósito**: WebGoat es una plataforma de entrenamiento de OWASP que no solo es vulnerable, sino que incluye lecciones teóricas integradas. WebWolf es su "compañero criminal", un servidor separado que simula un servidor de correo del atacante o un sitio para recibir archivos robados.
    *   **Lo mejor**: Es el más académico del grupo, guiándote paso a paso por las lecciones del OWASP Top 10.

### 🐳 Explicación de los Contenedores Utilizados
Este laboratorio utiliza la tecnología de **Docker** para encapsular cada aplicación en un entorno controlado y seguro:
*   **Aislamiento**: Cada servicio corre en su propio contenedor (como una "minicomputadora" virtual) con sus propias librerías (PHP, Node.js, Java). Esto evita conflictos y protege tu sistema operativo principal.
*   **Red Interna (`websec_network`)**: Los contenedores están conectados a una red virtual privada. Solo los puertos definidos en la columna "Puerto Host" son accesibles desde tu navegador en `localhost`, manteniendo el resto del tráfico privado entre ellos.
*   **Persistencia (Volúmenes)**: Se han configurado volúmenes para que tu progreso (bases de datos, configuraciones, retos completados) no se pierda al apagar o reiniciar los contenedores.
*   **Puertos**: El "Puerto Host" es la "puerta" que abres en tu PC. El "Puerto Contenedor" es el puerto interno donde escucha la aplicación dentro de Docker. Por ejemplo, aunque DVWA corre internamente en el puerto `80`, tú accedes por el `8001`.

---

## 🛠️ Gestión y Mantenimiento

Utiliza estos comandos desde la terminal para gestionar tu laboratorio:

*   **Verificar estado**: `docker-compose ps` (espera a que todos digan `healthy` o `Up`).
*   **Reiniciar servicios**: `docker-compose restart`.
*   **Ver logs en vivo**: `docker-compose logs -f`.
*   **Detener el lab**: `docker-compose down`.
*   **Limpiar TODO (incluyendo progreso/datos)**: `docker-compose down -v`.

---

## 🎯 Guía de Hackeo Paso a Paso (Laboratorio Práctico)

Esta sección detalla cómo explotar las vulnerabilidades presentes en el laboratorio de forma guiada y pedagógica.

### 1. SQL Injection (Inyección SQL)
**Definición Técnica**: Ocurre cuando un atacante puede interferir en las consultas que una aplicación realiza a su base de datos. Al insertar código SQL malicioso en un campo de entrada, el atacante puede engañar al servidor para que ejecute comandos no autorizados (como ver datos de otros usuarios, borrar tablas o saltarse la autenticación).

**Ejercicio Práctico (Nivel Principiante)**:
*   **Objetivo**: Extraer información de la base de datos que no debería ser pública.
*   **Servicio**: DVWA (`http://localhost:8001`)
*   **Configuración Obligatoria (Nivel de Seguridad)**: 
    Para que estos ejercicios funcionen, debes configurar DVWA en modo **Low**:
    1.  Inicia sesión (`admin` / `password`).
    2.  En el menú lateral izquierdo, haz clic en el botón **"DVWA Security"**.
    3.  En el desplegable, selecciona **"Low"**.
    4.  Presiona el botón **"Submit"**.
    5.  Verás que el nivel actual cambió a "low" en la parte inferior del menú.

*   **Paso a paso**:
    1.  Haz clic en el menú **"SQL Injection"** (en la barra lateral).
    2.  En el campo de texto "User ID", introduce el siguiente payload: `%' OR '1'='1`
    3.  Presiona el botón "Submit".
*   **¿Qué ocurrió?**: La consulta SQL original era algo como `SELECT first_name, last_name FROM users WHERE user_id = '$id'`. Al inyectar `' OR '1'='1`, la consulta se convierte en `WHERE user_id = '%' OR '1'='1'`, lo cual es siempre **verdadero**. Esto fuerza a la base de datos a devolver todos los registros existentes.

### 2. Bypass de Autenticación (SQL Injection Avanzado)
**Definición Técnica**: Es una variante de la Inyección SQL donde el objetivo no es extraer datos, sino manipular la lógica de validación de credenciales para obtener acceso al sistema sin una contraseña válida.

**Ejercicio Práctico**:
*   **Objetivo**: Entrar en una cuenta de administrador sin conocer la contraseña.
*   **Servicio**: OWASP Juice Shop (`http://localhost:8002`)
*   **Paso a paso**:
    1.  Ve a la página de Login: [http://localhost:8002/#/login](http://localhost:8002/#/login)
    2.  En el campo **Email**, introduce: `admin@juice-sh.op'--`
    3.  En el campo **Password**, escribe cualquier cosa (ej. `12345`).
    4.  Presiona "Log in".
*   **¿Qué ocurrió?**: El símbolo `'` cierra la cadena del email en la consulta SQL del servidor. Los caracteres `--` (o `#` en algunos casos) inician un **comentario**, lo que hace que el resto de la consulta (donde se comprueba la contraseña) sea ignorada por el motor de la base de datos. El sistema te deja entrar simplemente porque el email existe.

### 3. Cross-Site Scripting (XSS) Reflejado
**Definición Técnica**: El XSS ocurre cuando una aplicación web incluye datos no confiables en una página web sin la validación o el escape adecuados. En la variante "Reflejada", el script malicioso se "rebota" en el servidor y se ejecuta inmediatamente en el navegador de la víctima.

**Ejercicio Práctico**:
*   **Objetivo**: Ejecutar código JavaScript malicioso en el navegador de la víctima.
*   **Servicio**: DVWA (`http://localhost:8001`)
*   **Paso a paso**:
    1.  Asegúrate de que el nivel de seguridad sigue en `low` (revisa el menú lateral).
    2.  Haz clic en el menú **"XSS (Reflected)"**.
    3.  En el campo de nombre, introduce: `<script>alert('Tu navegador ha sido hackeado (demo)')</script>`
    4.  Presiona "Submit".
*   **¿Qué ocurrió?**: La aplicación toma lo que escribes y lo "refleja" directamente en el código HTML de la página sin limpiarlo. El navegador interpreta las etiquetas `<script>` como código real y ejecuta la función `alert()`. En un ataque real, esto podría usarse para robar cookies de sesión.

### 4. Inyección de Comandos (Command Injection)
**Definición Técnica**: Es una vulnerabilidad que permite a un atacante ejecutar comandos arbitrarios del sistema operativo (OS) en el servidor que ejecuta la aplicación. Esto ocurre cuando la aplicación pasa datos del usuario (como una IP para un ping) directamente a una shell del sistema sin filtrado.

**Ejercicio Práctico**:
*   **Objetivo**: Ejecutar comandos del sistema operativo del servidor a través de la web.
*   **Servicio**: DVWA (`http://localhost:8001`)
*   **Paso a paso**:
    1.  Ve al menú **"Command Injection"**.
    2.  En el campo de IP, introduce una dirección normal: `127.0.0.1` (verás que hace un ping).
    3.  Ahora, introduce el siguiente payload: `127.0.0.1; whoami`
    4.  Presiona "Submit".
*   **¿Qué ocurrió?**: La aplicación concatena tu entrada directamente a un comando de sistema (ej. `shell_exec('ping ' . $ip)`). El carácter `;` le indica a la terminal del servidor que el primer comando terminó y que debe ejecutar el siguiente (`whoami`). Esto permite a un atacante tomar control total del servidor.

### 5. Exposición de Datos Sensibles (Sensitive Data Exposure)
**Definición Técnica**: Ocurre cuando una aplicación web no protege adecuadamente información confidencial, como datos financieros, contraseñas o archivos de configuración del sistema. No siempre es un "hackeo" activo, sino a menudo un error de configuración donde directorios privados son accesibles públicamente.

**Ejercicio Práctico**:
*   **Objetivo**: Encontrar archivos confidenciales que el desarrollador olvidó proteger.
*   **Servicio**: OWASP Juice Shop (`http://localhost:8002`)
*   **Paso a paso**:
    1.  Explora la barra de navegación y entra en el menú lateral.
    2.  Ve a la sección **"About Us"**.
    3.  Busca enlaces o menciones a términos y condiciones.
    4.  Alternativamente, intenta acceder directamente a: `http://localhost:8002/ftp`
    5.  Intenta descargar algún archivo con extensión `.json` o `.pdf` que parezca privado.
*   **¿Qué ocurrió?**: Muchos servidores tienen directorios "olvidados" (como `/ftp`, `/backup`, `/admin`) que no están correctamente protegidos por controles de acceso. En Juice Shop, el directorio FTP está abierto y permite ver archivos que contienen configuraciones o datos de la arquitectura de la aplicación.

---

## ⚠️ Consideraciones de Seguridad
*   Este entorno es **estrictamente educativo**.
*   Las aplicaciones son **deliberadamente vulnerables**. No las expongas a redes públicas ni a Internet.
*   Se recomienda usar **WSL 2** como motor de Docker para una experiencia fluida.

---
*Diseñado por la IA Junie para la formación práctica en ciberseguridad.*
