# Despliegue en la Nube

El despliegue en la nube es una etapa crítica en el ciclo de vida del desarrollo de software, ya que permite que la aplicación esté disponible y accesible para los usuarios finales. En este proyecto, se utilizará **Amazon Web Services (AWS)** como plataforma de nube, aprovechando sus servicios robustos y escalables. La implementación se realizará mediante **AWS CloudFormation**, una herramienta que permite modelar y aprovisionar recursos de AWS de manera automatizada y reproducible.

## AWS CloudFormation

AWS CloudFormation es un servicio que facilita la gestión de infraestructura como código (IaC), permitiendo describir los recursos de AWS necesarios mediante archivos de plantilla en formato YAML o JSON. Esto asegura consistencia en los despliegues y simplifica la gestión de cambios en la infraestructura.

## Arquitectura de Despliegue

La arquitectura de despliegue propuesta incluye los siguientes componentes:

- **Amazon Elastic Container Service (ECS)**: Servicio de orquestación de contenedores para ejecutar la aplicación Dockerizada.
- **AWS Fargate**: Motor de cómputo sin servidor para contenedores que elimina la necesidad de administrar servidores.
- **Amazon Elastic Container Registry (ECR)**: Repositorio para almacenar las imágenes Docker de la aplicación.
- **Amazon Virtual Private Cloud (VPC)**: Red virtual aislada donde se desplegarán los recursos.
- **Load Balancer**: Para distribuir el tráfico entrante entre las instancias de la aplicación.

## Pasos para el Despliegue

### 1. Configuración del Entorno AWS

- **Credenciales de AWS**: Asegúrese de tener una cuenta de AWS con las credenciales necesarias (Access Key ID y Secret Access Key).
- **Configuración del CLI de AWS**: Instale y configure la AWS CLI en su entorno local o en el servidor de CI/CD.

```bash
aws configure
```

### 2. Construcción y Publicación de la Imagen Docker

- **Construir la imagen Docker**:
```bash
docker build -t your_microservice:latest .
```

- **Crear un repositorio en ECR**:
```bash
aws ecr create-repository --repository-name your_microservice --region us-east-1
```

- **Autenticarse en ECR**:
```bash
$(aws ecr get-login --no-include-email --region us-east-1)
```

- **Etiquetar y subir la imagen a ECR**:
```bash
docker tag your_microservice:latest <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/your_microservice:latest
docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/your_microservice:latest
```

### 3. Preparación de la Plantilla de CloudFormation
Cree un archivo YAML llamado cloudformation-template.yaml que describa los recursos necesarios para el despliegue.

Ejemplo de cloudformation-template.yaml
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Despliegue de YourMicroservice en AWS utilizando ECS y Fargate

Resources:
  YourMicroserviceCluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: your-microservice-cluster

  YourMicroserviceTaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      Family: your-microservice-task
      Cpu: '256'
      Memory: '512'
      NetworkMode: awsvpc
      RequiresCompatibilities:
        - FARGATE
      ExecutionRoleArn: arn:aws:iam::<aws_account_id>:role/ecsTaskExecutionRole
      ContainerDefinitions:
        - Name: your-microservice-container
          Image: <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/your_microservice:latest
          PortMappings:
            - ContainerPort: 8080
              Protocol: tcp

  YourMicroserviceService:
    Type: AWS::ECS::Service
    Properties:
      Cluster: !Ref YourMicroserviceCluster
      ServiceName: your-microservice-service
      TaskDefinition: !Ref YourMicroserviceTaskDefinition
      LaunchType: FARGATE
      DesiredCount: 2
      NetworkConfiguration:
        AwsvpcConfiguration:
          AssignPublicIp: ENABLED
          Subnets:
            - subnet-xxxxxxxx
            - subnet-xxxxxxxx
          SecurityGroups:
            - sg-xxxxxxxx
      LoadBalancers:
        - ContainerName: your-microservice-container
          ContainerPort: 8080
          TargetGroupArn: !Ref YourMicroserviceTargetGroup

  YourMicroserviceTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: your-microservice-tg
      Port: 8080
      Protocol: HTTP
      VpcId: vpc-xxxxxxxx
      TargetType: ip
      HealthCheckIntervalSeconds: 30
      HealthCheckProtocol: HTTP
      HealthCheckTimeoutSeconds: 5
      HealthyThresholdCount: 5
      UnhealthyThresholdCount: 2
      Matcher:
        HttpCode: '200'

  YourMicroserviceLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: your-microservice-lb
      Subnets:
        - subnet-xxxxxxxx
        - subnet-xxxxxxxx
      SecurityGroups:
        - sg-xxxxxxxx
      Scheme: internet-facing
      Type: application

  YourMicroserviceListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref YourMicroserviceLoadBalancer
      Port: 80
      Protocol: HTTP
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref YourMicroserviceTargetGroup
```

**Nota**: Reemplace <aws_account_id>, subnet-xxxxxxxx, sg-xxxxxxxx, y vpc-xxxxxxxx con los valores correspondientes de su cuenta y entorno de AWS.

### 4. Despliegue de la Plantilla con CloudFormation
- **Crear el stack de CloudFormation**:
```bash
aws cloudformation create-stack --stack-name your-microservice-stack --template-body file://cloudformation-template.yaml --capabilities CAPABILITY_NAMED_IAM
```
- **Monitorear el estado del stack**:
```bash
aws cloudformation describe-stacks --stack-name your-microservice-stack
```
- **Actualizar el stack (si es necesario)**:
```bash
aws cloudformation update-stack --stack-name your-microservice-stack --template-body file://cloudformation-template.yaml --capabilities CAPABILITY_NAMED_IAM
```

### 5. Validación del Despliegue

- **Obtener la URL del Load Balancer**:
```bash
aws cloudformation describe-stacks --stack-name your-microservice-stack --query "Stacks[0].Outputs"
```

- **Probar la aplicación**:
Acceda a la URL obtenida en el paso anterior y verifique que la aplicación esté funcionando correctamente.

## Consideraciones de Seguridad
- **Roles y Políticas de IAM**: Asegúrese de que los roles utilizados tengan los permisos mínimos necesarios siguiendo el principio de privilegio mínimo.
- **Almacenamiento de Credenciales**: No almacene credenciales en el código o en repositorios. Utilice AWS Secrets Manager o AWS Systems Manager Parameter Store para gestionar secretos de forma segura.
- **Cifrado de Datos**: Considere habilitar el cifrado en tránsito y en reposo donde sea aplicable.


## Buenas Prácticas
- **Infraestructura como Código (IaC)**: Mantenga las plantillas de CloudFormation y scripts de despliegue versionados junto con el código de la aplicación.
- **Automatización de Despliegues**: Integre el despliegue con su sistema de CI/CD para asegurar despliegues consistentes y reproducibles.
- **Monitoreo y Logging**: Configure Amazon CloudWatch para monitorear métricas y logs de la aplicación y la infraestructura.
- **Escalabilidad**: Configure Auto Scaling en el servicio de ECS para ajustar automáticamente la capacidad en respuesta a la demanda.
- **Gestión de Costos**: Supervise el uso de recursos y optimice la configuración para controlar costos.

## Recursos Adicionales
- [Documentación de AWS CloudFormation](https://docs.aws.amazon.com/es_es/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [Mejores Prácticas para AWS CloudFormation](https://docs.aws.amazon.com/es_es/AWSCloudFormation/latest/UserGuide/best-practices.html)
- [Despliegue de Contenedores en Amazon ECS](https://docs.aws.amazon.com/es_es/AmazonECS/latest/developerguide/Welcome.html)
- [AWS Fargate](https://docs.aws.amazon.com/es_es/AmazonECS/latest/developerguide/AWS_Fargate.html)