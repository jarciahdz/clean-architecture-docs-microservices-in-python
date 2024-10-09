# Configuración del Entorno

Para garantizar un entorno de desarrollo consistente y facilitar la implementación y despliegue del microservicio, es esencial configurar adecuadamente el entorno. Esta sección proporciona instrucciones detalladas para preparar el entorno de desarrollo, incluyendo la instalación de dependencias, configuración de variables de entorno y uso de contenedores.

## Requisitos Previos

- **Python 3.9 o superior**: Asegúrese de tener instalada la versión adecuada de Python.
- **Pip**: El gestor de paquetes de Python para instalar las dependencias.
- **Virtualenv** (opcional pero recomendado): Para crear un entorno virtual aislado.
- **Docker** (opcional pero recomendado): Para facilitar el despliegue y ejecución en contenedores.

## Instalación de Dependencias

Es recomendable utilizar un entorno virtual para aislar las dependencias del proyecto.

### Creación de un Entorno Virtual

### venv
```bash

# Crear un entorno virtual
python -m venv venv

# Activar el entorno virtual (Unix/Mac)
source venv/bin/activate

# Activar el entorno virtual (Windows)
venv\Scripts\activate
```

### virtualenv
```bash
# Instalar virtualenv si no lo tiene instalado
pip install virtualenv

# Crear un entorno virtual
virtualenv venv

# Activar el entorno virtual (Unix/Mac)
source venv/bin/activate

# Activar el entorno virtual (Windows)
venv\Scripts\activate
```

## Instalación de Paquetes
Con el entorno virtual activado, instale las dependencias listadas en requirements.txt:

```bash
pip install -r requirements.txt
```

## Configuración de Variables de Entorno

Las variables de entorno permiten configurar parámetros sin alterar el código fuente. Cree un archivo .env en la raíz del proyecto para definir variables sensibles o específicas del entorno.

### Ejemplo de Archivo .env
```env
# Configuración de la base de datos
DB_HOST=localhost
DB_PORT=5432
DB_USER=usuario
DB_PASSWORD=contraseña
DB_NAME=nombre_base_datos

# Configuración del entorno
ENVIRONMENT=development

# Configuración de logging
LOG_LEVEL=DEBUG
```

Asegúrese de que el archivo .env esté incluido en el archivo .gitignore para evitar subir información sensible al repositorio.

## Configuración de Logging
El archivo config/logging.conf contiene la configuración para el sistema de logging de la aplicación. Ajuste los parámetros según las necesidades de monitoreo y depuración.

### Estructura de logging.conf
```ini
[loggers]
keys=root,app

[handlers]
keys=consoleHandler,fileHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=WARNING
handlers=consoleHandler

[logger_app]
level=DEBUG
handlers=consoleHandler,fileHandler
qualname=app
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=simpleFormatter
args=(sys.stdout,)

[handler_fileHandler]
class=FileHandler
level=INFO
formatter=simpleFormatter
args=('logs/app.log', 'a')

[formatter_simpleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=
```

## Uso de Docker

Para facilitar la ejecución y despliegue de la aplicación en diferentes entornos, puede utilizar Docker para contenerizar el microservicio.

### Construcción de la Imagen Docker
```bash
docker build -t your_microservice:latest .
```

### Ejecución del Contenedor
```bash
docker run -d -p 8080:8080 --env-file .env your_microservice:latest
```

## Uso de Docker Compose
Si su aplicación depende de otros servicios (por ejemplo, una base de datos), puede utilizar docker-compose.yml para orquestar múltiples contenedores.yaml

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    env_file:
      - .env
    depends_on:
      - db
  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=${DB_NAME}
    ports:
      - "5432:5432"
```

Para iniciar los servicios:

```bash
docker-compose up -d
```

## Configuración del Archivo settings.py
El archivo config/settings.py centraliza la configuración de la aplicación, leyendo variables de entorno y proporcionando valores predeterminados cuando sea necesario.

### Ejemplo de settings.py
```python
import os
from dotenv import load_dotenv

# Cargar variables de entorno desde .env
load_dotenv()

class Settings:
    # Configuración de la base de datos
    DB_HOST = os.getenv('DB_HOST', 'localhost')
    DB_PORT = int(os.getenv('DB_PORT', 5432))
    DB_USER = os.getenv('DB_USER', 'user')
    DB_PASSWORD = os.getenv('DB_PASSWORD', 'password')
    DB_NAME = os.getenv('DB_NAME', 'database')

    # Configuración del entorno
    ENVIRONMENT = os.getenv('ENVIRONMENT', 'development')

    # Configuración de logging
    LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')

settings = Settings()
```

## Recomendaciones Adicionales
- **Mantenga las Dependencias Actualizadas**: Revise periódicamente el archivo requirements.txt y actualice los paquetes para incluir parches de seguridad y mejoras.
- **Seguridad de Variables Sensibles**: No comparta archivos que contengan información sensible y utilice herramientas como Vault o servicios de gestión de secretos para entornos de producción.
- **Consistencia en Entornos**: Use herramientas de automatización y contenedores para asegurar que los entornos de desarrollo, pruebas y producción sean lo más similares posible.