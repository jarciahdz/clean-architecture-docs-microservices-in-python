---
sidebar_position: 3
sidebar_label: Infraestructura
---

# Capa de Infraestructura

La **Capa de Infraestructura** proporciona implementaciones concretas para las interfaces definidas en las capas superiores, manejando detalles técnicos y de comunicación con sistemas externos. Esta capa es donde se integra la aplicación con tecnologías específicas, como bases de datos, servicios web, sistemas de mensajería, entre otros.

## Responsabilidades

- **Implementación de Repositorios**: Provee las clases que interactúan directamente con los sistemas de persistencia, cumpliendo con los contratos definidos en las interfaces del dominio.

- **Servicios Externos**: Maneja la comunicación con APIs externas, colas de mensajes y otros servicios fuera del control de la aplicación.

- **Gestión de Recursos Técnicos**: Configura y administra conexiones, sesiones y otros recursos necesarios para la infraestructura.

## Principios Clave

- **Cumplimiento de Contratos**: Las clases de esta capa deben adherirse estrictamente a las interfaces y contratos definidos en el dominio y la aplicación.

- **Aislamiento de Detalles Técnicos**: Los detalles de implementación deben estar contenidos en esta capa, evitando que las capas superiores dependan de tecnologías específicas.

- **Facilidad de Sustitución**: Al implementar interfaces, es posible reemplazar componentes de infraestructura sin afectar el resto del sistema (por ejemplo, cambiar de una base de datos SQL a NoSQL).

## Estructura de Directorios

```plaintext
app/
├── infrastructure/
    ├── repositories/
    │   └── your_repository_impl.py
    └── services/
        └── external_service.py
```

## Ejemplo

### your_repository_impl.py

```python
from app.domain.repositories.your_repository_interface import YourRepositoryInterface

class YourRepositoryImpl(YourRepositoryInterface):
    def __init__(self, db_connection):
        self.db_connection = db_connection

    def get_by_id(self, entity_id):
        # Implementación específica utilizando la base de datos
        result = self.db_connection.query("SELECT * FROM entities WHERE id = ?", (entity_id,))
        if result:
            return YourEntity(**result[0])
        return None

    def save(self, entity):
        # Implementación específica para guardar la entidad
        self.db_connection.execute("INSERT INTO entities (...) VALUES (...)", entity.to_dict())
```

## Servicio Externo

### external_service.py

```python

class ExternalService:
    def __init__(self, api_client):
        self.api_client = api_client

    def send_data(self, data):
        response = self.api_client.post("/external-endpoint", json=data)
        return response.status_code == 200

```

## Beneficios

- **Modularidad**: Facilita la gestión y organización de componentes relacionados con la infraestructura.

- **Flexibilidad Tecnológica**: Permite cambiar o actualizar tecnologías subyacentes sin modificar las capas de dominio o aplicación.

- **Responsabilidad Clara**: Centraliza el manejo de detalles técnicos, lo que simplifica el mantenimiento y la resolución de problemas.

## Capas

- [Dominio](dominio.md)
- [Aplicación](presentacion.md)
- [Infraestructura](infraestructura.md)
- [Presentación](presentacion.md)
