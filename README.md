# Conectar React con Node usando Docker Compose

Este documento contiene los pasos necesarios para conectar una aplicación React con un servidor Node.js utilizando Docker Compose.

## Pasos iniciales

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
   - Nota: La IP `127.xxx...` no es válida, ya que es una IP de loopback.

4. **Conectar con SSH desde tu equipo local:**
   ```bash
   ssh usuario@ip_del_servidor
   ```
   Si es la primera vez, confirma la huella digital escribiendo `yes`.

---

## Crear las carpetas y subdirectorios del proyecto

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
     cd ~/despliegue/node
     git clone https://github.com/frperezp/next-productivity-API
     ```
   - Para React (frontend):
     ```bash
     cd ~/despliegue/react
     git clone https://github.com/frperezp/next-productivity-app
     ```

4. **Instalar Docker Compose:**
   ```bash
   sudo apt install docker-compose
   ```

---

## Configurar Docker Compose

Crea el archivo `docker-compose.yaml` en la carpeta principal `despliegue` con el siguiente contenido:

```yaml
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
FROM node:18-alpine

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .
EXPOSE 3000

CMD ["npm", "start"]
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

## Levantar los contenedores

Ejecuta el siguiente comando desde la carpeta `despliegue`:

```bash
docker-compose up --build
```

Esto construirá las imágenes y levantará los contenedores con las configuraciones especificadas.

---

## Verificación

1. **Verificar que los servicios están corriendo:**
   ```bash
   docker ps
   ```

2. **Acceder a los servicios:**
   - Frontend (React): [http://localhost:3000](http://localhost:3000)
   - Backend (Node.js): [http://localhost:5000](http://localhost:5000)

3. **Conexión remota:** Si accedes desde otro equipo, usa la IP pública del servidor en lugar de `localhost`.

---

## Configuración avanzada: Docker Buildx

### Instalación de Docker Buildx

1. **Verificar si está instalado:**
   ```bash
   docker buildx version
   ```

2. **Instalación (si es necesario):**
   ```bash
   mkdir -p ~/.docker/cli-plugins
   curl -L https://github.com/docker/buildx/releases/download/v0.8.0/buildx-v0.8.0-linux-amd64 -o ~/.docker/cli-plugins/docker-buildx
   chmod +x ~/.docker/cli-plugins/docker-buildx
   ```

3. **Habilitar Buildx:**
   ```bash
   docker buildx create --use
   ```

4. **Construir imágenes multiplataforma:**
   ```bash
   docker buildx build --platform linux/amd64,linux/arm64 -t nombre_imagen .
   ```

---

## Estructura del Proyecto

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

---

## Solución de problemas comunes

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

