# Pasos para conectar React con Node usando Docker Compose

Pasos necesarios para conectar una aplicación React con un servidor Node.js utilizando Docker Compose.

## Primeros pasos

### Configurar acceso SSH en el servidor

1. **Habilitar SSH:** Durante la instalación del servidor, asegúrate de permitir conexiones SSH. Si no está habilitado:
   ```bash
   sudo apt update
   sudo apt install openssh-server
   sudo systemctl enable ssh
   sudo systemctl start ssh
   ```

2. **Configurar red en VirtualBox:** Si utilizas VirtualBox, configura la red en modo "Adaptador puente" para que la máquina virtual esté en la misma red que el equipo anfitrión.

3. **Obtener la IP del servidor:**
   - Usa el comando `ip addr` en el servidor.
   - Nota: La IP `127.xxx...` no es válida.

4. **Conectar con SSH desde tu equipo local:**
   ```bash
   ssh usuario@ip_del_servidor
   ```
   Si es la primera vez, confirma la huella digital escribiendo `yes`.

---
## Finalizados los primeros pasos...
## Crear las carpetas y subdirectorios del proyecto
*Todas las direciones y nombres debes cambiarlos según tu configuración de carpetas, lo que te muestro yo es como lo organice yo*

1. **Estructura inicial:**
   ```bash
   mkdir -p ~/despliegue/react ~/despliegue/node
   ```

2. **Asignar permisos:**
   ```bash
   sudo chown -R $(whoami):$(whoami) ~/despliegue
   sudo chmod -R u+rwx ~/despliegue
   ```

3. **Clonar los repositorios:**
   - Para Node.js (API):
     ```bash
     mkdir ~/despliegue/node
     cd ~/despliegue/node
     sudo git clone https://github.com/frperezp/next-productivity-API
     ```
   - Para React (frontend):
     ```bash
     cd ~/despliegue
     mkdir react
     cd react
     sudo git clone https://github.com/frperezp/next-productivity-app
     ```

4. **Instalar Docker Compose:**
   ```bash
   cd ~/despliegue
   sudo apt install docker-compose
   ```
   
5. **Renombrar el archivo Dockerfile**
   ```bash
   mv dockerfile Dockerfile
   ```


---

## Configurar Docker Compose

Crea el archivo `docker-compose.yaml` en la carpeta principal `despliegue` con el siguiente contenido:

```yaml
# Version of Docker-compose
# https://www.youtube.com/watch?v=0B2raYYH2fE

version: '3.9'

services:
  react-app:
    build:
      context: ./react/next-productivity-app
      dockerfile: Dockerfile
    container_name: react-app
    ports:
      - '3000:3000'
    stdin_open: true
    networks:
      - pro-app

  api-node:
    build:
      context: ./node/next-productivity-API
      dockerfile: Dockerfile
    container_name: api-node
    ports:
      - '5000:5000'
    networks:
      - pro-app
    volumes:
      - ./node/next-productivity-API/tasks.json:/app/tasks.json

networks:
  pro-app:
    driver: bridge
```

---

## Crear los Dockerfiles

### Para React
Crea el archivo `Dockerfile` dentro de la carpeta `react/next-productivity-app`:

```dockerfile
# Usa la imagen base oficial de node:18-alpine para la aplicación Node.js
FROM node:18-alpine

# Establece el directorio de trabajo dentro del contenedor
WORKDIR /app

# Copia el package.json y package-lock.json para aprovechar la caché de Docker
# y solo ejecutar npm install si los archivos de dependencias cambian
COPY package.json ./
COPY package-lock.json ./

# Instala las dependencias de la aplicación
RUN npm install

# Copia el resto del código fuente de la aplicación al contenedor
COPY . .

# Establece las variables de entorno necesarias (puedes cambiar esta URL según se>ENV REACT_APP_BASE_URL=http://192.168.1.179:5000/tasks

```

### Para Node.js
Crea el archivo `Dockerfile` dentro de la carpeta `node/next-productivity-API`:

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .
EXPOSE 5000

CMD ["npm", "start"]
```

---
## Verificación

1. **Verificar si los servicios están en ejecución:**
   ```bash
   docker ps
   ```
2. **Acceder a la aplicación:**

   Si todo está correctamente configurado y los contenedores están levantados, puedes     acceder a los servicios desde el navegador:
    - Frontend (React): [http://localhost:3000](http://localhost:3000)
    - Backend (Node.js): [http://localhost:5000](http://localhost:5000)

3. **Accede al servicio de forma remota:**
  
   Si necesitas acceder a tu servidor desde otro equipo en la red, usa la IP pública      de tu servidor en lugar de localhost (por ejemplo: ip_de_tu_ubuntu_server:3000).

## Levantar los contenedores

Ejecuta el siguiente comando desde la carpeta `despliegue`:

*Si ya tienes las imágenes construidas, puedes iniciar los contenedores sin reconstruirlos:*

•	Si las imágenes ya han sido construidas previamente y no has realizado cambios en los archivos del proyecto, puedes iniciar los contenedores sin necesidad de reconstruir las imágenes. Para ello, utiliza el siguiente comando:
  ```bash
   docker-compose up -d
   ```

*Si realizas cambios en los archivos, asegúrate de reconstruir las imágenes:*

•	Si has hecho modificaciones en el código o en la configuración, es necesario reconstruir las imágenes antes de levantar los contenedores. Para hacerlo, utiliza el siguiente comando:
   ```bash
   docker-compose up --build -d
   ```

Esto construirá las imágenes y levantará los contenedores con las configuraciones especificadas.

---

## Configuración avanzada: Docker Buildx

### Instalación de Docker Buildx

1. **Verificar si está instalado:**
   ```bash
   docker buildx version
   ```

   *Si ves una versión de buildx en la salida, significa que ya está instalado y puedes saltarte los siguientes pasos.*

2. **Instalación (si es necesario):**

   Asegúrate de tener la última versión de Docker:
  
   ```bash
   sudo apt update
   sudo apt install -y docker.io
   ```

   Habilitar el soporte de Docker Buildx de forma manual:
   
   ```bash
   mkdir -p ~/.docker/cli-plugins
   curl -L https://github.com/docker/buildx/releases/download/v0.8.0/buildx-v0.8.0-       linux-amd64 -o ~/.docker/cli-plugins/docker-buildx
   chmod +x ~/.docker/cli-plugins/docker-buildx
   ```

4. **Habilitar Buildx:**
   ```bash
   docker buildx create --use
   ```

5. **Verificar instalación:**
   ```bash
   docker buildx version
   ```

*Si todo fue bien, deberias de ver la versión de Buildx, a continuación tienes dos opciones:*

6a. **Construir imágenes multiplataforma:**
   ```bash
   docker buildx build --platform linux/amd64,linux/arm64 -t nombre_imagen .
   ```

6b. **Empujar la imagen a un repositorio:**

Si quieres empujar la imagen a un repositorio como Docker Hub, puedes usar el siguiente comando:

   ```bash
   docker buildx build --platform linux/amd64,linux/arm64 -t nombre_imagen .
   ```
El final --push: envía la imagen al repositorio especificado (asegúrate de haber iniciado sesión en Docker Hub o cualquier otro registro de contenedores).

7. **Verificar la imagen construida:**

Para ver todas las imágenes que has construido, puedes usar el comando:
   ```bash
   docker images
   ```
Y para ver los "builders" disponibles:
   ```bash
   docker buildx ls
   ```
---

## Estructura del Proyecto

Tree de como deben quedar las carpetas y docs si lo hiciste igual que yo

```plaintext
despliegue/                            # Directorio principal del proyecto
├── node/                               # Carpeta para el backend (Node.js)
│   ├── next-productivity-API/          # Repositorio clonado para la API de Node.js
│   │   ├── Dockerfile                  # Dockerfile para la API de Node.js
│   │   ├── tasks.json                  # Archivo de tareas de la API
│   │   └── server.js                   # Código del servidor de la API
│
├── react/                              # Carpeta para el frontend (React)
│   ├── next-productivity-app/          # Repositorio clonado para la aplicación React
│   │   ├── Dockerfile                  # Dockerfile para la aplicación React
│   │   ├── src/                        # Archivos fuente de React
│   │   │   ├── components/             # Componentes React
│   │   │   ├── App.js                  # Componente principal
│   │   │   └── index.js                # Archivo de entrada de React
│
├── docker-compose.yaml                 # Archivo de configuración de Docker Compose
```
**Ejecutar:**
   ```bash
   docker-compose up --build
   ```
*Si todo está correcto deberia salir:*



---

## Solución de problemas

1. **Error de timeout:**
   ```bash
   export COMPOSE_HTTP_TIMEOUT=200
   ```

2. **Reiniciar contenedores con cambios:**
   ```bash
   docker-compose up --build -d
   ```

---

¡Y listo! Tu aplicación React y Node.js debería estar funcionando correctamente con Docker Compose.

