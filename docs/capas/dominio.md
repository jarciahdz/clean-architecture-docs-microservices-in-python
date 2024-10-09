# Capa de Dominio

La **Capa de Dominio** es el núcleo central de la arquitectura y representa las reglas de negocio y las entidades fundamentales del sistema. Esta capa es completamente independiente de detalles de implementación externos, como frameworks, bases de datos o interfaces de usuario, lo que garantiza que los cambios en esas áreas no afecten la lógica esencial de la aplicación.

## Responsabilidades

- **Entidades (Entities)**: Son objetos del dominio que encapsulan atributos y comportamientos coherentes con las reglas de negocio. Representan conceptos clave del sistema, como usuarios, productos o transacciones.

- **Interfaces de Repositorio**: Definen contratos para la persistencia y recuperación de entidades sin especificar detalles técnicos. Estas interfaces permiten que la capa de aplicación interactúe con los datos de manera abstracta.

- **Reglas de Negocio**: Contienen la lógica y validaciones que son intrínsecas al dominio, asegurando la integridad y consistencia de los datos y procesos.

## Principios Clave

- **Independencia Tecnológica**: La capa de dominio no depende de ninguna tecnología externa, lo que facilita su reutilización y portabilidad.

- **Modelo Rico**: Las entidades no son simples estructuras de datos; pueden contener lógica relevante que garantiza que siempre se encuentren en un estado válido.

- **Abstracción**: Las interfaces permiten definir contratos claros sin exponer detalles de implementación, promoviendo la inversión de dependencias.

## Estructura de Directorios

```plaintext
app/
├── domain/
    ├── entities/
    │   └── your_entity.py
    └── repositories/
        └── your_repository_interface.py
```

## Ejemplo

### Entidad

#### your_entity.py

```python
class YourEntity:
    def __init__(self, attribute1, attribute2):
        self.attribute1 = attribute1
        self.attribute2 = attribute2
        self.validate()

    def validate(self):
        if not self.attribute1:
            raise ValueError("attribute1 es obligatorio")
        # Otras validaciones y lógica de negocio
```

### Interfaz de Repositorio

#### your_repository_interface.py

```python
from abc import ABC, abstractmethod

class YourRepositoryInterface(ABC):
    @abstractmethod
    def get_by_id(self, entity_id):
        pass

    @abstractmethod
    def save(self, entity):
        pass
```

## Beneficios

- **Testabilidad Mejorada**: Al no depender de componentes externos, las pruebas unitarias pueden centrarse en la lógica de negocio pura.

- **Mantenibilidad**: Los cambios en tecnologías o frameworks no afectan esta capa, reduciendo el impacto de modificaciones.

- **Claridad Conceptual**: Facilita la comprensión del dominio del problema al estar aislado y bien definido.


## Capas
- [Dominio](dominio.md)
- [Aplicación](presentacion.md)
- [Infraestructura](infraestructura.md)
- [Presentación](presentacion.md)


---

© 2024 Línea De Conocimientos Machine Learning E Inteligencia Artificial- BANCOLOMBIA.