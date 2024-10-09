# Capa de Presentación

La **Capa de Presentación** es la responsable de gestionar la interacción entre el usuario o cliente y la aplicación. Esta capa expone la funcionalidad del sistema a través de interfaces como APIs RESTful, interfaces gráficas de usuario (GUIs) o interfaces de línea de comandos (CLIs).

## Responsabilidades

- **Controladores**: Manejan las solicitudes entrantes, coordinan la ejecución de los casos de uso y preparan las respuestas adecuadas.

- **Configuración de Frameworks**: Inicializa y configura los frameworks utilizados para gestionar las interfaces de usuario o protocolos de comunicación.

- **Validación y Formateo**: Realiza validaciones de datos de entrada y formatea las respuestas según sea necesario (por ejemplo, JSON).

## Principios Clave

- **Dependencia en la Capa de Aplicación**: Los controladores utilizan los casos de uso definidos en la capa de aplicación para ejecutar la lógica de negocio.

- **Aislamiento de la Lógica de Negocio**: No deben contener lógica de negocio; su función es coordinar y delegar.

- **Adaptabilidad**: Permite cambiar o agregar nuevas interfaces de usuario sin modificar las capas internas de la aplicación.

## Estructura de Directorios

```plaintext
app/
├── presentation/
    ├── controllers/
    │   └── your_controller.py
    └── frameworks/
        └── flask_app.py
```

## Ejemplos
### Controlador
#### your_controller.py

```python

from app.application.use_cases.your_use_case import YourUseCase
from app.infrastructure.repositories.your_repository_impl import YourRepositoryImpl

class YourController:
    def __init__(self):
        repository = YourRepositoryImpl(db_connection)
        self.use_case = YourUseCase(repository)

    def handle_request(self, request_data):
        # Validación de datos de entrada
        input_data = self.validate_request(request_data)
        # Ejecución del caso de uso
        result = self.use_case.execute(input_data)
        # Preparación de la respuesta
        response = self.format_response(result)
        return response

    def validate_request(self, request_data):
        # Implementar validaciones necesarias
        pass

    def format_response(self, result):
        # Formatear el resultado para la respuesta
        pass

```

### Configuración del Framework
#### flask_app.py

```python

from flask import Flask, request, jsonify
from app.presentation.controllers.your_controller import YourController

app = Flask(__name__)
controller = YourController()

@app.route('/endpoint', methods=['POST'])
def endpoint():
    request_data = request.get_json()
    response = controller.handle_request(request_data)
    return jsonify(response), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)

```


## Beneficios
- **Interfaz Consistente**: Proporciona una interfaz uniforme para que los clientes interactúen con la aplicación.

- **Separación de Preocupaciones**: Aísla las responsabilidades de presentación y protocolo de comunicación de la lógica de negocio.

- **Escalabilidad**: Facilita la adición de nuevos puntos de entrada o la adaptación a nuevos canales de comunicación.

## Capas
- [Dominio](dominio.md)
- [Aplicación](presentacion.md)
- [Infraestructura](infraestructura.md)
- [Presentación](presentacion.md)
