# Integración Continua y Despliegue Continuo (CI/CD)

La implementación de una estrategia robusta de **Integración Continua (CI)** y **Despliegue Continuo (CD)** es esencial para garantizar la calidad, confiabilidad y eficiencia en el ciclo de vida del desarrollo de software. Este proyecto utiliza **Azure DevOps** como plataforma principal para orquestar las canalizaciones de CI/CD, y considera despliegues tanto en **Azure** como en **AWS** para aprovechar las fortalezas de ambas nubes.

## Configuración de Azure DevOps

### 1. Creación del Proyecto en Azure DevOps

- Acceda al portal de [Azure DevOps](https://dev.azure.com/) y autentíquese con sus credenciales.
- Cree una nueva organización o utilice una existente.
- Genere un nuevo proyecto seleccionando **"Crear proyecto"** y proporcionando un nombre y descripción adecuados.

### 2. Repositorio de Código Fuente

- Importe el código fuente del proyecto al repositorio Git integrado en Azure DevOps o enlace un repositorio externo, como GitHub.
- Verifique que la estructura del proyecto y todos los archivos necesarios (por ejemplo, `Dockerfile`, `docker-compose.yml`, `requirements.txt`, etc.) estén presentes en el repositorio.

## Configuración de la Canalización de Integración Continua (CI)

La canalización de CI automatiza la construcción y las pruebas del código cada vez que se produce un cambio en el repositorio.

### 1. Creación de la Canalización

- Navegue a la sección **"Pipelines"** en Azure DevOps y seleccione **"Crear Pipeline"**.
- Seleccione el repositorio que contiene el código fuente.
- Opte por **"Inicio rápido"** para utilizar una plantilla básica o elija **"Archivo YAML"** para definir la canalización mediante un archivo `azure-pipelines.yml`.

### 2. Definición del Archivo `azure-pipelines.yml`

Cree un archivo llamado `azure-pipelines.yml` en la raíz del proyecto con el siguiente contenido:

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  PYTHON_VERSION: '3.9'

steps:
  - task: UsePythonVersion@0
    inputs:
      versionSpec: '$(PYTHON_VERSION)'
    displayName: 'Configurar Python $(PYTHON_VERSION)'

  - script: |
      python -m pip install --upgrade pip
      pip install -r requirements.txt
    displayName: 'Instalar dependencias'

  - script: |
      pytest tests/ --junitxml=test-results.xml
    displayName: 'Ejecutar pruebas'
  
  - task: PublishTestResults@2
    inputs:
      testResultsFormat: 'JUnit'
      testResultsFiles: '**/test-results.xml'
    displayName: 'Publicar resultados de pruebas'

  - task: PublishBuildArtifacts@1
    inputs:
      pathToPublish: '$(System.DefaultWorkingDirectory)'
      artifactName: 'drop'
    displayName: 'Publicar artefactos de compilación'

```

### Explicación de la Canalización
- **trigger**: Configura la canalización para que se ejecute en las ramas especificadas (main y develop).
- **pool**: Especifica el agente de compilación a utilizar (ubuntu-latest).
- **variables**: Define variables globales, como la versión de Python.
- **steps**: Lista de tareas a ejecutar:
    - **UsePythonVersion**: Configura la versión de Python en el agente.
    - **Instalar dependencias**: Actualiza pip e instala las dependencias del proyecto.
    - **Ejecutar pruebas**: Ejecuta las pruebas utilizando pytest y genera un archivo de resultados en formato JUnit.
    - **Publicar resultados de pruebas**: Publica los resultados para que puedan ser visualizados en Azure DevOps.
    - **Publicar artefactos de compilación**: Guarda los artefactos generados durante la compilación para su uso posterior en la canalización de CD.


### 3. Configuración de Variables de Entorno
En la sección de variables de la canalización, configure las variables necesarias, como credenciales o configuraciones específicas del entorno.
Utilice variables secretas para manejar información sensible, asegurándose de que estén protegidas y no se expongan en los logs.

## Configuración de la Canalización de Despliegue Continuo (CD)
La canalización de CD automatiza el proceso de despliegue de la aplicación a los entornos correspondientes (desarrollo, pruebas, producción).

### 1. Creación de una Release Pipeline
Navegue a la sección "Releases" en Azure DevOps y seleccione "Nueva canalización".
Elija "Empty Job" para comenzar desde cero o seleccione una plantilla según sus necesidades.
### 2. Definición de las Etapas de Despliegue
- **Desarrollo**: Primera etapa donde se despliega la aplicación en un entorno de desarrollo para pruebas iniciales.
- **Pruebas**: Etapa donde se realizan pruebas más exhaustivas, incluyendo pruebas de integración y aceptación.
- **Producción**: Etapa final donde la aplicación se despliega al entorno de producción.
### 3. Configuración de Tareas de Despliegue
En esta sección se abordará la configuración de las tareas de despliegue para ambos entornos en la nube: Azure y AWS.

#### Despliegue en Azure
Para desplegar la aplicación en Azure, se utilizarán servicios como Azure Container Instances (ACI) o Azure Kubernetes Service (AKS), dependiendo de las necesidades.

##### Ejemplo de Tarea de Despliegue en Azure
### 4. Configuración de Gates y Aprobaciones
Establezca gates y aprobaciones manuales entre las etapas para controlar el flujo de despliegue.
Configure notificaciones para informar a los equipos relevantes sobre el progreso y estado de los despliegues.
```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Conexión a Azure'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      # Configuración de variables
      IMAGE_NAME=your_microservice
      IMAGE_TAG=$(Build.BuildId)
      ACR_NAME=youracrname
      RESOURCE_GROUP=your-resource-group
      CONTAINER_GROUP=your-container-group

      # Inicio de sesión en ACR
      az acr login --name $ACR_NAME

      # Construcción y etiquetado de la imagen
      docker build -t $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG .

      # Envío de la imagen a ACR
      docker push $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG

      # Despliegue en ACI
      az container create \
        --resource-group $RESOURCE_GROUP \
        --name $CONTAINER_GROUP \
        --image $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG \
        --registry-login-server $ACR_NAME.azurecr.io \
        --registry-username $(ACR_USERNAME) \
        --registry-password $(ACR_PASSWORD) \
        --dns-name-label your-dns-label \
        --ports 8080
  displayName: 'Desplegar en Azure Container Instances'
```

#### Despliegue en AWS
Para el despliegue en AWS, se pueden utilizar servicios como Amazon Elastic Container Service (ECS) o Amazon Elastic Kubernetes Service (EKS).

##### Configuración de Credenciales AWS
Antes de ejecutar tareas en AWS, es necesario configurar las credenciales de acceso. Esto se puede hacer mediante variables de entorno o utilizando el servicio de Azure Key Vault para almacenar secretos.

##### Ejemplo de Tarea de Despliegue en AWS
```yaml
- task: AWSCLI@1
  inputs:
    awsCredentials: 'Conexión a AWS'
    awsRegionName: 'us-east-1'
    scriptType: 'inline'
    inlineScript: |
      # Configuración de variables
      IMAGE_NAME=your_microservice
      IMAGE_TAG=$(Build.BuildId)
      AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
      ECR_REPO_NAME=your-ecr-repo

      # Inicio de sesión en ECR
      aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com

      # Construcción y etiquetado de la imagen
      docker build -t $IMAGE_NAME .

      # Etiquetado de la imagen para ECR
      docker tag $IMAGE_NAME:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG

      # Envío de la imagen a ECR
      docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$ECR_REPO_NAME:$IMAGE_TAG

      # Despliegue en ECS o EKS (se requiere configuración adicional)
      # Por ejemplo, actualizar una tarea de ECS con la nueva imagen
  displayName: 'Desplegar en AWS Elastic Container Service'
```

## Integración con Azure Key Vault

### Integración con Azure Key Vault
Para manejar de forma segura las credenciales y secretos necesarios durante el proceso de despliegue, se integra Azure Key Vault en las canalizaciones.

Configuración de Azure Key Vault
- Cree un Azure Key Vault en su suscripción de Azure.
- Agregue los secretos necesarios, como contraseñas, claves de API y otras credenciales sensibles.
- Configure las políticas de acceso para permitir que Azure DevOps Pipeline acceda al Key Vault.

#### Uso en la Canalización
Agregue una tarea en el azure-pipelines.yml para obtener los secretos del Key Vault:

```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Conexión a Azure'
    KeyVaultName: 'your-key-vault-name'
    SecretsFilter: '*'
    RunAsPreJob: true
  displayName: 'Descargar secretos desde Azure Key Vault'
```
Con esta configuración, los secretos estarán disponibles como variables en la canalización y pueden ser utilizados en los scripts de despliegue.

## Buenas Prácticas
- **Versionado Semántico**: Utilice un esquema de versionado consistente para las builds y despliegues.
- **Principio de Menor Privilegio**: Configure los permisos de acceso de las canalizaciones y servicios con el mínimo necesario.
- **Validación en Pull Requests**: Configure políticas de rama que requieran la aprobación y el paso de las pruebas antes de fusionar cambios.
- **Despliegue Azul-Verde o Canary**: Considere estrategias avanzadas de despliegue para minimizar el impacto en producción.
- **Seguridad de Credenciales**: Nunca almacene credenciales sensibles en texto plano o en el repositorio de código. Utilice servicios como Azure Key Vault o AWS Secrets Manager.
- **Automatización Completa**: Busque que el proceso de CI/CD sea completamente automatizado, reduciendo la intervención manual y minimizando errores.
- **Monitoreo y Alertas**: Integre servicios de monitoreo como Azure Monitor o AWS CloudWatch para supervisar el rendimiento y estado de la aplicación después del despliegue.
- **Despliegue Multinube**: Al considerar despliegues en Azure y AWS, asegúrese de que su aplicación y procesos sean compatibles con ambos entornos, adoptando prácticas de infraestructura como código cuando sea posible.
- **Infraestructura como Código**: Utilice herramientas como Terraform o Azure Resource Manager (ARM) Templates para definir y gestionar la infraestructura de forma declarativa.

## Recursos Adicionales
- [Documentación de Azure DevOps](https://learn.microsoft.com/es-es/azure/devops/?view=azure-devops)
- [Prácticas recomendadas para CI/CD en Azure](https://learn.microsoft.com/en-us/azure/devops/pipelines/architectures/devops-pipelines-baseline-architecture?view=azure-devops)
- [Integración de Azure Key Vault con Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/azure-key-vault?view=azure-devops&tabs=classic)
- [Despliegue de Contenedores en AWS](https://docs.aws.amazon.com/ecs/)