---
sidebar_position: 2
sidebar_label: Aplicación
---

# Capa de Aplicación

La **Capa de Aplicación** actúa como orquestadora de los casos de uso y coordina la interacción entre la capa de dominio y las demás capas del sistema. Su principal objetivo es implementar la lógica específica de la aplicación sin incorporar detalles de infraestructura o presentación.

## Responsabilidades

- **Casos de Uso (Use Cases)**: Representan secuencias de acciones que la aplicación puede realizar, encapsulando la lógica de procesos y flujos de trabajo.

- **Coordinación de Entidades y Repositorios**: Gestiona cómo las entidades del dominio interactúan con los repositorios y otros servicios necesarios.

- **Validaciones y Reglas de Aplicación**: Implementa reglas que son propias de la aplicación pero que no pertenecen al dominio puro.

## Principios Clave

- **Independencia de Infraestructura**: No debe contener código relacionado con bases de datos, redes u otros sistemas externos.

- **Orientada a Casos de Uso**: Cada caso de uso representa una funcionalidad específica y completa desde el punto de vista del usuario o sistema.

- **Facilidad de Testeo**: Al mantenerse aislada de detalles externos, facilita la creación de pruebas unitarias para validar la lógica de aplicación.

## Estructura de Directorios

```plaintext
app/
├── application/
    └── use_cases/
        └── your_use_case.py
```

## Ejemplo

### Caso de Uso

#### your_use_case.py

```python

class YourUseCase:
    def __init__(self, repository):
        self.repository = repository

    def execute(self, input_data):
        # Validaciones de entrada
        if not input_data:
            raise ValueError("Los datos de entrada son obligatorios")

        # Interacción con el dominio
        entity = self.repository.get_by_id(input_data['id'])
        if not entity:
            raise EntityNotFoundException("Entidad no encontrada")

        # Lógica de aplicación
        entity.process()
        self.repository.save(entity)

        # Retornar resultado o estado
        return entity
```

## Beneficios

- **Claridad en la Lógica de Negocio**: Al separar los casos de uso, se facilita la comprensión y mantenimiento de la funcionalidad de la aplicación.

- **Reutilización**: Los casos de uso pueden ser utilizados por diferentes interfaces de usuario o sistemas externos sin duplicar código.

- **Flexibilidad**: Permite modificar flujos de trabajo sin afectar las capas de dominio o infraestructura.

## Capas

- [Dominio](dominio.md)
- [Aplicación](presentacion.md)
- [Infraestructura](infraestructura.md)
- [Presentación](presentacion.md)
