# Pruebas

Las pruebas son un componente esencial en el desarrollo de software de alta calidad. Garantizan que la aplicación funcione según lo esperado, facilitando la detección temprana de errores y mejorando la confiabilidad y mantenibilidad del sistema. Este proyecto incorpora una estrategia de pruebas integral que incluye pruebas unitarias y de integración.

## Organización de las Pruebas

El código de las pruebas se encuentra organizado en el directorio `tests/`, siguiendo una estructura que refleja la organización de la aplicación:

```plaintext
tests/
├── unit/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── presentation/
└── integration/
    ├── repositories/
    └── services/
```

## Pruebas Unitarias
Las pruebas unitarias validan el funcionamiento correcto de componentes individuales en aislamiento, como funciones, métodos o clases, sin depender de componentes externos o sistemas. Estas pruebas se ubican en el directorio tests/unit/ y están organizadas por capas para reflejar la estructura de la aplicación.

### Herramientas Utilizadas
- **Pytest**: Un marco de pruebas potente y flexible que simplifica la escritura de pruebas unitarias.
- **Unittest**:: El módulo estándar de Python para pruebas unitarias, utilizado en casos donde se requiera compatibilidad con herramientas existentes.

### Ejemplo de Prueba Unitaria

#### tests/unit/domain/entities/test_your_entity.py
```python
import pytest
from app.domain.entities.your_entity import YourEntity

def test_your_entity_initialization():
    entity = YourEntity(attribute1='value1', attribute2='value2')
    assert entity.attribute1 == 'value1'
    assert entity.attribute2 == 'value2'

def test_your_entity_validation():
    with pytest.raises(ValueError):
        YourEntity(attribute1=None, attribute2='value2')

```

## Pruebas de Integración
Las pruebas de integración verifican la interacción entre diferentes componentes del sistema, asegurando que funcionen correctamente en conjunto. Estas pruebas pueden involucrar bases de datos, servicios externos o módulos de infraestructura.

### Herramientas Utilizadas
- **Pytest**: También utilizado para pruebas de integración por su capacidad para manejar fixtures y configuraciones complejas.
- **Docker Compose**:: Para levantar servicios dependientes como bases de datos durante las pruebas.

### Ejemplo de Prueba de Integración
#### tests/integration/repositories/test_your_repository_impl.py

```python
import pytest
from app.infrastructure.repositories.your_repository_impl import YourRepositoryImpl
from app.domain.entities.your_entity import YourEntity
from config.settings import settings

@pytest.fixture
def repository():
    # Configuración de la conexión a la base de datos de prueba
    db_connection = setup_test_db_connection()
    return YourRepositoryImpl(db_connection)

def test_repository_save_and_get(repository):
    entity = YourEntity(attribute1='value1', attribute2='value2')
    repository.save(entity)
    retrieved_entity = repository.get_by_id(entity.id)
    assert retrieved_entity == entity
```

## Cobertura de Pruebas
Es recomendable medir la cobertura de las pruebas para identificar áreas del código que no están siendo verificadas. Herramientas como Coverage.py pueden integrarse con Pytest para generar reportes detallados.

### Generación de Reporte de Cobertura
```bash
pytest --cov=app tests/
```

### Ejecución de las Pruebas
Para ejecutar todas las pruebas, utilice el siguiente comando en la raíz del proyecto:

```bash
pytest tests/
```

Puede especificar directorios o archivos específicos:

- **Ejecutar solo pruebas unitarias**:
```bash
pytest tests/unit/
```

- **Ejecutar solo pruebas de integración**::
```bash
pytest tests/integration/
```

## Buenas Prácticas
- **Escribir Pruebas Primero (TDD)**: Considerar la adopción del Desarrollo Dirigido por Pruebas para mejorar la calidad y diseño del código.
- **Pruebas Aisladas**: Asegurarse de que las pruebas unitarias no dependan del estado de otras pruebas o de componentes externos.
- **Nombres Descriptivos**: Utilizar nombres claros y descriptivos para las funciones de prueba, facilitando la comprensión del propósito de cada una.
- **Uso de Fixtures**: Emplear fixtures para configurar y limpiar el entorno de pruebas de manera eficiente.

## Integración Continua
Integrar las pruebas en el pipeline de Integración Continua (CI) es fundamental para detectar problemas de manera temprana. Configure el sistema CI para ejecutar las pruebas en cada commit o pull request.

## Recomendaciones Adicionales
- **Mocking**: Utilizar técnicas de mocking para simular componentes externos y aislar el código bajo prueba.
- **Pruebas de Regresión**: Mantener y actualizar las pruebas al realizar cambios en el código para prevenir la reintroducción de errores.
- **Documentación de Pruebas**: Documentar casos de prueba complejos y escenarios especiales para facilitar el mantenimiento y colaboración.
