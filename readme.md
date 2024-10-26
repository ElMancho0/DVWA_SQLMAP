# DVWA y Kali Linux con SQLMap - Docker Setup

Este proyecto utiliza Docker Compose para levantar dos servicios: 
1. **DVWA (Damn Vulnerable Web Application)**: Una aplicación web vulnerable para pruebas de seguridad.
2. **Kali Linux**: Un sistema operativo enfocado en pruebas de penetración, que puedes usar para atacar DVWA usando herramientas como SQLMap.

## Requisitos

Asegúrate de tener Docker y Docker Compose instalados en tu máquina antes de proceder. Si no los tienes, sigue estos pasos:

- [Instalar Docker](https://docs.docker.com/get-docker/)
- [Instalar Docker Compose](https://docs.docker.com/compose/install/)

## Instrucciones de uso

### 1. Levantar los servicios

Para levantar ambos servicios (DVWA y Kali Linux), simplemente ejecuta:

```bash
docker-compose up -d
```

Esto iniciará los dos contenedores:

- **DVWA** estará disponible en [http://localhost](http://localhost)
- **Kali Linux** estará disponible como un contenedor interactivo donde puedes correr comandos.

### 2. Acceder al contenedor de Kali Linux

Para ingresar al contenedor de Kali Linux, utiliza el siguiente comando:

```bash
docker exec -it kali /bin/bash
```

Esto abrirá una terminal dentro del contenedor de Kali.

### 3. Acceder a DVWA

Abre tu navegador web y visita [http://localhost](http://localhost) para acceder a la interfaz de DVWA. Puedes usar el nombre de usuario `admin` y la contraseña `password` para iniciar sesión. Una vez dentro tiene que presionar el boton para crear la base de datos

### 4. Usar SQLMap

Para usar sqlmap debe usar el comando que vimos al inicio para abrir el contenor de kali linux una vez dentro el sqlmap ya está instalado por lo que solo necesitan usar un comando como el siguiente para acceder a sqlmap. Pero para poder atacar el sitio, la seguridad del mismo pide tener una sesion por lo que necesitamos unas cookies que sirvan como credenciales.

```bash
sqlmap -u "http://dvwa/login.php" --forms --batch
```

### 5. Obtener las cookies


Para obtener las credenciales los pasos son:
- Iniciar sesion en dvwa, puede ser con admin/password
- Entrar en modo desarrollador del sitio web
- Entre el menu, ir a storage y despues a cookies
- Entre las cookies de localhost hay una llamada "PHPSESSID" copian el nombre y el valor para usarla en el comando de la siguiente forma

```bash
sqlmap -u "http://dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie="PHPSESSID=3jnfo12s7lim6st1vflr6lc5; security=low" --dbs
```

## Apagar los servicios

Cuando hayas terminado, puedes detener los contenedores con el siguiente comando:

```bash
docker-compose down
```

Esto apagará y eliminará los contenedores.
