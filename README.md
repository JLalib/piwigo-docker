# 🖼️ Piwigo en Docker - Galería de fotos autohospedada

[![GitHub](https://img.shields.io/badge/GitHub-Piwigo%2Fpiwigo--docker-blue?logo=github)](https://github.com/Piwigo/piwigo-docker)
[![Docker](https://img.shields.io/badge/Docker-piwigo%2Fpiwigo-blue?logo=docker)](https://hub.docker.com/r/piwigo/piwigo)
[![License](https://img.shields.io/badge/License-GPL--2.0-orange)](https://www.gnu.org/licenses/gpl-2.0.html)

## 📋 Descripción general

Piwigo es una galería de fotos autohospedada con licencia GPL-2.0 que te permite subir, organizar y compartir tus imágenes en tu propio servidor. Se despliega con Docker Compose junto a una base de datos MariaDB, manteniendo todos los datos en carpetas del host (`./piwigo-data`) para facilitar copias de seguridad y migraciones. El contenedor oficial incluye nginx, PHP-FPM y MariaDB en dos servicios separados conectados mediante una red bridge interna.

## ✨ Características principales

- **Álbumes y permisos**: Organiza fotos en álbumes y controla el acceso por usuario o grupo
- **Temas y plugins**: Personaliza la apariencia y amplía funcionalidades desde el panel de administración
- **Datos en carpetas**: Archivos, base de datos y scripts viven en `./piwigo-data` sin volúmenes Docker anónimos
- **Scripts propios**: Ejecuta comandos o instala paquetes de Alpine antes de arrancar con `user.sh`
- **Actualizaciones sencillas**: Al actualizar la imagen, los ficheros nuevos se copian sobre los existentes
- **Base de datos opcional**: Compose alternativo (`compose-nodb.yaml`) para usar MySQL/MariaDB externo

## 📋 Requisitos del sistema

- Docker y Docker Compose instalados
- Un puerto libre en el host (configurable vía `piwigo_port`)
- Servidor o NAS con acceso SSH
- Opcional: Proxy inverso para dominio/subdominio con HTTPS

## 🐳 Instalación

### Paso 1: Descarga el compose oficial

```bash
mkdir piwigo
cd piwigo
curl -O https://raw.githubusercontent.com/Piwigo/piwigo-docker/main/compose.yaml
curl -O https://raw.githubusercontent.com/Piwigo/piwigo-docker/main/.env.example
cp .env.example .env
```

### Paso 2: Completa el fichero `.env`

```bash
# Puerto del host. Con 127.0.0.1:8080 solo accede el propio servidor
piwigo_port=8080

# Contraseña del usuario de la base de datos. Cámbiala
db_user_password=cambia-esta-contrasena

# Zona horaria
timezone=Europe/Madrid

# Usuario y grupo dueños de la carpeta piwigo-data
PIWIGO_UID=1000
PIWIGO_GID=1000
```

> **Importante**: Docker salta las reglas del firewall por diseño. Si pones solo `8080`, el puerto queda abierto a toda tu red o a Internet si reenvías el router. Para limitarlo, usa `piwigo_port=127.0.0.1:8080` y publica la galería a través de un proxy.
>
> **UID y GID**: Averigua los tuyos con `id -u` e `id -g` y ponlos en `PIWIGO_UID` y `PIWIGO_GID`. Así tu usuario podrá gestionar las fotos desde el host sin problemas de permisos.

### Paso 3: Arranca el contenedor

```bash
docker compose up -d
# Revisa los logs si algún servicio no arranca
docker compose logs -f piwigo-main
```

### Paso 4: Completa la instalación

Abre `http://IP_DEL_SERVIDOR:8080`. Piwigo detecta que no hay nada instalado y te lleva a la página de instalación.

**Datos de la base de datos:**
- Host: `piwigo-db:3306`
- Usuario: `piwigodb_user`
- Base de datos: `piwigodb`
- Contraseña: la que pusiste en `db_user_password`
- Prefijo de tablas: `piwigo_`

Pulsa "Start Install" y crea después tu cuenta de administrador.

## ⚙️ Configuración

1. **Puerto de exposición** (`piwigo_port`): Define cómo se publica el puerto (ej. `8080` o `127.0.0.1:8080` para solo localhost)
2. **Credenciales de BD** (`db_user_password`): Contraseña segura para el usuario de la base de datos
3. **Zona horaria** (`timezone`): Ajusta a tu zona (ej. `Europe/Madrid`, `America/Mexico_City`)
4. **Permisos de archivos** (`PIWIGO_UID`/`PIWIGO_GID`): UID/GID del usuario que gestionará los archivos en el host
5. **Base de datos externa** (opcional): Usa `compose-nodb.yaml` y elimina `db_user_password` del `.env`

## 🚀 Primeros pasos

1. Completa la instalación web y crea tu cuenta de administrador
2. Sube un álbum de prueba y comprueba que las miniaturas se generan correctamente
3. Crea usuarios o grupos y asigna álbumes privados según necesites
4. Haz una copia de seguridad de `./piwigo-data` antes de cualquier actualización

## 💡 Casos de uso

- **Fotos familiares**: Álbumes privados para cada persona y uno compartido
- **Portfolio**: Galería pública con tema propio en tu dominio
- **Archivo de viajes**: Organiza miles de fotos en álbumes por fecha o lugar

## 🔒 Acceso remoto seguro

Para exponer Piwigo de forma segura:

- Usa `piwigo_port=127.0.0.1:8080` en `.env` para no abrir puertos en la red
- Configura un proxy inverso (nginx, Traefik, Caddy) con HTTPS y autenticación
- Si publicas en subpath (ej. `tudominio.com/galeria`), el proxy debe enviar la cabecera `X-Forwarded-Prefix` con esa ruta

## 🛠️ Gestión y mantenimiento

### Ver estado
```bash
docker compose ps
```

### Ver logs
```bash
docker compose logs -f piwigo-main
```

### Copia de seguridad
```bash
# Detén el contenedor antes de copiar
docker compose down
rsync -r --delete-before ./piwigo-data/ ./piwigo-data.bck/
docker compose up -d
```

### Actualizar
```bash
# Haz primero la copia de seguridad
docker compose down
docker compose pull
docker compose up -d
```

> **Versiones**: La imagen `latest` es la que usa el compose oficial. Si prefieres fijar una versión, cambia la etiqueta de `piwigo/piwigo` en `compose.yaml` por la que quieras.

### Detener Piwigo
```bash
docker compose down
```

## 📝 Licencia

GNU General Public License v2.0 - Ver [LICENSE](https://www.gnu.org/licenses/gpl-2.0.html)

---

> **Stack técnico**: Contenedor app: Alpine + nginx + PHP-FPM (usuario compartido) | Base de datos: MariaDB LTS (`docker.io/library/mariadb:lts`) | Red: bridge interna `piwigo-network` | Imagen: `piwigo/piwigo:latest` en Docker Hub
>
> **Referencias oficiales**: [Piwigo GitHub](https://github.com/Piwigo/Piwigo) · [piwigo-docker](https://github.com/Piwigo/piwigo-docker) · [Guía oficial Docker](https://github.com/Piwigo/piwigo-docker/blob/main/README.md) · [Imagen Docker Hub](https://hub.docker.com/r/piwigo/piwigo)
>
> 📖 **Guía completa**: [Cómo instalar Piwigo en Docker - Galería de fotos autohospedada](https://genbyte.blogspot.com/2026/11/como-instalar-piwigo-en-docker-galeria.html)