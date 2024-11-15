# P5-DNS-Compose
**Práctica: Configuración de DNS con Docker Compose**  
**Por: Martín Álvarez Comesaña**  

---

### Propuesta: Configuración de la imagen bind9 utilizando Docker Compose

#### Organización
Primero, crearemos un directorio para organizar nuestro proyecto:  
```bash
mkdir P5_DNS_Docker_Compose && cd P5_DNS_Docker_Compose
```

El proyecto se estructurará en un archivo principal, llamado `docker-compose.yml`, que es un archivo de configuración que define y ejecuta aplicaciones multi-contenedor en Docker. Este archivo especificará cómo construir y ejecutar los contenedores.

Además, se crearán dos directorios principales que se usarán como volúmenes, siguiendo las indicaciones de configuración de la imagen. Estos directorios serán `/etc/bind` y `/var/cache/bind`, que pueden crearse con el siguiente comando:  
```bash
mkdir -p ./etc/bind ./var/cache/bind
```

El directorio `/etc` se usará para la configuración de BIND, mientras que `/var` se reservará para el espacio de trabajo de BIND.

---

#### Permisos
Antes de crear los archivos de configuración, otorgaremos los permisos adecuados para los directorios:  
```bash
sudo chown -R 100:100 ./etc/bind/ ./var/cache/bind
```

Esto asegura que los directorios sean propiedad del usuario y grupo con ID `100`, permitiendo que BIND funcione correctamente dentro de un contenedor Docker.

---

#### Archivo `docker-compose.yml`
Crearemos este archivo dentro del directorio `P5_DNS_Docker_Compose`:  
```bash
nano ./docker-compose.yml
```

El archivo incluirá una configuración básica basada en las recomendaciones de la imagen:  

```yaml
services:
  bind9: 
    image: internetsystemsconsortium/bind9:9.18
    container_name: Practica5_bind9
    ports:
      - 54:53/udp
      - 54:53/tcp
      - 127.0.0.1:953:953/tcp
    volumes:
      - ./etc/bind:/etc/bind
      - ./var/cache/bind:/var/cache/bind
    restart: always
```

**Notas importantes:**
- Los volúmenes se especifican primero con la ruta del host (`./etc/bind`), seguida de la ruta dentro del contenedor (`/etc/bind`).
- Presta atención a la indentación: usa espacios en lugar de tabulaciones.

---

#### Archivo `named.conf`
Este archivo es fundamental para configurar el servidor BIND. Basándonos en las configuraciones de la imagen, añadiremos lo siguiente:  

```bash
options {
    directory "/var/cache/bind";
};
```

Esto le indica al servidor que almacene sus datos de caché en el directorio `/var/cache/bind`, que ya configuramos previamente.

---

#### Verificación
Para comprobar que todo funciona correctamente, situémonos en el directorio donde se encuentra el archivo `docker-compose.yml` y ejecutemos:  
```bash
docker compose up -d
```

Deberíamos ver algo como:  
```
[+] Running 2/2
 ✔ Network <nombre_de_red>  Created          0.1s
 ✔ Container Practica5_bind9  Started       0.8s
```

Luego, identificaremos la IP del contenedor:  
```bash
docker network inspect <nombre_de_red>
```

En la respuesta, busca el apartado `Containers` y la línea `IPv4Address`. Por ejemplo:  
```json
"IPv4Address": "172.18.0.2/16"
```

Usando esta IP, probaremos el servidor DNS:  
```bash
dig @172.18.0.2
```

---

#### Errores comunes
1. **Puerto 53 en uso:**  
   Si ves un error como:  
   ```bash
   Error response from daemon: driver failed programming external connectivity
   ```  
   Cambia el puerto `53` por otro, como `54`.  

2. **Error en el archivo `docker-compose.yml`:**  
   Si aparece algo como:  
   ```bash
   Additional property volume is not allowed
   ```  
   Revisa que la palabra clave sea `volumes` y no `volume`.  

3. **Errores en el archivo `named.conf`:**  
   Si el contenedor falla con el código `0`, revisa la sintaxis del archivo. Por ejemplo, asegúrate de no olvidar el `;` al final de cada línea.

--- 

Esta práctica está diseñada para afianzar conceptos sobre configuración de DNS y manejo de contenedores Docker.
