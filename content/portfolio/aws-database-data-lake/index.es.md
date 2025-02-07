---
title: "Construcción de un Lago de Datos en AWS"
date: 2024-12-20
description: "Este fue un proyecto para uno de mis cursos de maestría."
tags: ["Datalake", "AWS"]
showDate: true
showWordCount: true
draft: false
---

![alt text](image-15.png)

# **Introducción**

Los datos son la base de la toma de decisiones en el mundo digital actual. Desde la detección de fraudes hasta la optimización de operaciones empresariales, las organizaciones dependen de pipelines de datos eficientes para ingerir, procesar y analizar grandes cantidades de información. Pero gestionar estos pipelines a gran escala requiere más que bases de datos tradicionales, se necesita una arquitectura robusta y escalable.

Este proyecto nació de un curso que tomé durante mi maestría en Analítica de Datos, específicamente en **Sistemas de Bases de Datos Avanzados**, donde nos propusimos diseñar y desplegar un **Data Lake en AWS**. ¿El objetivo? Construir un sistema capaz de integrar datos de múltiples fuentes de manera fluida mientras garantizábamos **eficiencia, automatización y escalabilidad**. En lugar de solo configurar servicios, me enfoqué en estructurar un pipeline que pudiera manejar desafíos de datos del mundo real, como procesar datos estructurados y semiestructurados, automatizar la ingestión y optimizar los costos.

En este blog, te contaré mi experiencia y a lo largo del camino, resaltaré lecciones clave aprendidas, decisiones arquitectónicas tomadas y mejoras que haría en futuras iteraciones.

---

# **Contexto y Antecedentes**

Para nuestro proyecto final, el instructor nos dio **dos opciones**:

1️⃣ **Opción A** → Desarrollar un **sistema de consultas avanzadas** que interactúe con varias bases de datos en la nube.  
2️⃣ **Opción B** → **Construir un Data Lake en AWS**, siguiendo una arquitectura de extremo a extremo demostrada en un video de referencia.

El video, [**Building Data Lakes on AWS**](https://youtu.be/LAovcGquDc4?si=XxqlKeQLFqRuHaIb), proporcionaba una guía paso a paso para configurar un Data Lake utilizando servicios como **AWS Glue, Athena y S3**. Sin embargo, el instructor dejó claro que **no era necesario replicar el video exactamente**, dándonos la flexibilidad de modificar la implementación según fuera necesario.

Decidimos optar por **Opción B**, pero queríamos **evitar AWS Glue** para eliminar costos durante el desarrollo. Esto requirió cambios adicionales, como reemplazar Glue por otros servicios de AWS y configurar un **programador** para automatizar el procesamiento de datos. Estas modificaciones introdujeron nuevos desafíos técnicos y aumentaron la complejidad del proceso de desarrollo.

![alt text](image-2.png)

---

# **Visión General del Proyecto**

En su núcleo, este proyecto consistió en **construir un Data Lake en la nube utilizando AWS**. Pero, ¿qué significa eso en la práctica? Aquí tienes un desglose a alto nivel:

- **Almacenamiento de Datos**: Un **bucket de AWS S3** sirve como el repositorio principal para los datos entrantes desde las bases de datos.  
- **Infraestructura de Base de Datos**: Se implementan múltiples bases de datos (MySQL, SQL Server y PostgreSQL) utilizando **Amazon RDS**.  
- **Gestión de Costos con CloudFormation**: Se utilizaron plantillas de CloudFormation para **desplegar y eliminar bases de datos a diario** durante el desarrollo. Este enfoque ayudó a minimizar los costos mientras el proyecto seguía evolucionando.  
- **Procesamiento ETL**: Las funciones de AWS Lambda se encargan de los procesos de **Extract, Transform, Load (ETL)** para limpiar y mover los datos.  
- **Orquestación**: **Amazon EventBridge** activa y programa las funciones Lambda para procesar los datos de manera eficiente.  
- **Seguridad**: AWS Secrets Manager garantiza un manejo seguro de las credenciales de las bases de datos.

### **Desafíos y Lecciones Aprendidas**

Como en cualquier proyecto técnico, este vino con su parte de desafíos:

- Configuración de conexiones seguras entre las instancias de RDS y las funciones Lambda.  
- Optimización de costos aprovechando los recursos de la **capa gratuita** de AWS.  
- Automatización de todo el flujo de trabajo mientras se mantenía la flexibilidad para futuros cambios.

Al final, tuve una arquitectura funcional de **Data Lake**, capaz de ingerir datos desde múltiples fuentes y prepararlos para análisis. No es perfecta, pero es una base sólida.

![alt text](image.png)

---


# **El Viaje Personal**

Cada proyecto tiene un lado técnico, pero siempre hay un viaje personal detrás del código. Este proyecto no fue la excepción.

## **Cómo Comenzó**

Configurar un **Data Lake en AWS** fue una gran oportunidad para trabajar con múltiples servicios de AWS de manera estructurada. Ya estaba familiarizado con AWS, pero este proyecto requería integrar múltiples servicios mientras mantenía los costos bajos. Fue la oportunidad perfecta para **aprender, romper cosas y descubrir cómo hacer que funcionen** en un escenario del mundo real.

La documentación de AWS es útil, pero a menudo se enfoca en **cómo** configurar los servicios en lugar de **por qué** ciertas decisiones son importantes. En lugar de simplemente seguir guías, probamos diferentes configuraciones para encontrar el mejor enfoque para nuestro proyecto. Esta experimentación práctica nos ayudó a perfeccionar nuestra arquitectura, solucionar problemas y tomar decisiones informadas.

## **Desafíos Clave y Cómo Los Superamos**

### **Problemas de Conectividad de Base de Datos**

Antes de lanzar AWS RDS, probamos la conectividad con nuestras bases de datos existentes en **Azure y MongoDB**. Estas pruebas tempranas nos ayudaron a verificar las interacciones entre las bases de datos, pero debido a las restricciones de tiempo, decidimos alojar todo en **Amazon RDS** para mantener la consistencia.

El siguiente desafío surgió al configurar la **red de VPC**. Ejecutar Lambda dentro de una VPC requería configurar grupos de seguridad, subredes y gateways NAT. Para depurar el acceso a la red, lanzamos una **instancia EC2 dentro de la VPC** y la usamos para probar las conexiones a RDS. Esto nos ayudó a identificar y ajustar rápidamente las reglas de seguridad, lo que permitió una comunicación fluida con la base de datos.

##### **Solución**
- Utilizamos una **instancia EC2 dentro de la VPC** para probar las conexiones antes de desplegar las funciones Lambda.  
- Configuramos **grupos de seguridad y enrutamiento de subredes** para permitir que las funciones Lambda accedieran a RDS de manera segura.  
- Aseguramos que los **roles IAM y los puntos finales de VPC** estuvieran configurados para una interacción eficiente con la base de datos.

---

### **Automatización de la Infraestructura con CloudFormation**

Tenía experiencia previa con **CloudFormation**, por lo que desplegar recursos a través de plantillas no fue un desafío. Sin embargo, el principal problema fue asegurarnos de que las **configuraciones de bases de datos funcionaran con las instancias de bajo nivel de AWS**. Algunos parámetros de RDS no eran compatibles con las instancias de bajo nivel, lo que causaba fallos en el despliegue.

##### **Solución**
- Ajustamos los **grupos de parámetros de la base de datos** para que coincidan con las limitaciones del nivel de instancia.  
- Desplegamos iterativamente pilas de CloudFormation para identificar **problemas de compatibilidad de recursos**.

Este enfoque nos permitió desplegar bases de datos rápidamente y **eliminarlas después del desarrollo diario** para mantener los costos bajos.

---

### **Procesamiento ETL y Programación de Eventos**

El principal desafío con AWS Lambda fue configurar **capas personalizadas** para manejar las consultas a la base de datos. Necesitábamos bibliotecas externas de Python, pero nos encontramos con limitaciones en la máquina local, límites de tamaño de capa y problemas de compatibilidad de bibliotecas.

Para resolverlo, usamos nuestra **instancia EC2** para construir la capa Lambda, asegurándonos de que todas las dependencias estuvieran correctamente empaquetadas. Luego, la capa se subió a **S3** y se vinculó a las funciones Lambda.

##### **Solución**
- Construimos la **capa Lambda** en una instancia EC2 para manejar las dependencias de paquetes.  
- Comprimimos y subimos la capa a **S3** para su fácil reutilización.  
- Ajustamos la **memoria y el tiempo de espera de las funciones Lambda** para mejorar el rendimiento.

Una vez que las funciones Lambda estaban funcionando de manera eficiente, las programamos utilizando **Amazon EventBridge** para automatizar el pipeline ETL.

---

## **Lo que Mejoraría la Próxima Vez**  

🔹 **Más automatización**: Aunque **CloudFormation** ayudó, incorporar **Terraform** podría proporcionar una mayor flexibilidad en la gestión de infraestructura.  
🔹 **Registro y monitoreo**: Agregar **alertas de AWS CloudWatch** mejoraría la visibilidad de fallos y el rendimiento del sistema.  

---

# **Profundización Técnica**  

Ahora que hemos cubierto el recorrido, vamos a sumergirnos en los detalles de cómo se construyó este Data Lake. Esta sección desglosará cada componente principal, desde **la ingestión de datos** hasta **el procesamiento y almacenamiento**, destacando configuraciones clave y mejores prácticas a lo largo del camino.  

---

## **Ingestión de Datos: Configuración de AWS S3 como el Data Lake**  

El primer paso para construir un Data Lake es definir dónde se almacenarán los datos. En este caso, **Amazon S3** sirve como la base, actuando como una solución de almacenamiento escalable y rentable.  

### **Creación del Bucket de S3**  
Para configurar el **bucket de Data Lake**, utilicé la consola de AWS y seguí las mejores prácticas:  

- **Deshabilité las ACLs** → Asegura que todos los objetos sean propiedad de mi cuenta.  
- **Bloqueé el acceso público** → Previene la exposición no deseada de datos.  
- **Activé la versión** → Mantiene versiones históricas de los objetos en caso de errores.  
- **Encriptación por defecto (AES-256)** → Protege los datos en reposo.  

```bash
aws s3api create-bucket --bucket utp-database-data-lake-project --region us-east-1
```

Una vez creado el bucket, se convirtió en el repositorio central para la ingestión de datos. **Todos los datos sin procesar de varias bases de datos (MySQL, SQL Server y PostgreSQL) fueron almacenados aquí antes de ser procesados.**  

![alt text](image-4.png)

---

## **Infraestructura como Código: AWS CloudFormation**  

Provisionar manualmente bases de datos y recursos de red es ineficiente, por lo que automatizé el proceso utilizando **AWS CloudFormation**. CloudFormation permite definir la infraestructura en una **plantilla YAML**, haciendo que los despliegues sean **repetibles y escalables**.  

### **Stack de Base de Datos (Instancias RDS)**  
Modifiqué una plantilla de CloudFormation proporcionada por AWS para desplegar tres instancias **Amazon RDS**:  

- **MySQL** (para transacciones de comercio electrónico)  
- **SQL Server** (para datos CRM)  
- **PostgreSQL** (para datos de gestión empresarial)  

Cada instancia de base de datos tenía la siguiente configuración:  

- **Tipo de instancia:** `db.t3.micro`  
- **Almacenamiento:** 20GB  
- **Seguridad:** Integración con el rol IAM y **AWS Secrets Manager** para el almacenamiento de credenciales  
- **Redes:** Subred privada con grupos de seguridad VPC  

```yaml
Resources:
  MySQLInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: mysql-instance
      DBName: db_mysql
      DBInstanceClass: db.t3.micro
      AllocatedStorage: 20
      Engine: MySQL
      EngineVersion: "8.0.39"
      MasterUsername: 
        Fn::Sub: "{{resolve:secretsmanager:arn:aws:secretsmanager:us-east-1:123456789:secret:utp/database/rds-ASDAS:SecretString:MySQLDBUsername}}"
```

Una vez desplegado, la sección de **Outputs de CloudFormation** proporcionó **puntos finales para conectar** a cada instancia de base de datos.  

![alt text](image-6.png)

---

## **Procesamiento de Datos con AWS Lambda (Pipelines ETL)**  

Extraer, transformar y cargar (ETL) datos de manera eficiente es crucial para un Data Lake bien funcionando. Se utilizó **AWS Lambda** para **automatizar la extracción de datos, procesar los datos crudos y almacenar versiones refinadas** en S3.  

### **Flujo de Trabajo ETL**  
Cada función Lambda era responsable de un paso diferente en el pipeline ETL:  

1. **Extraer datos de RDS**  
2. **Transformar los registros crudos en formatos estructurados (Parquet, CSV)**  
3. **Cargar los datos procesados nuevamente en S3**  

```python
import os
import json
import logging
from io import BytesIO

import pymysql
import boto3
from botocore.exceptions import ClientError
import pandas as pd

# Configuración de logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Inicialización de clientes de AWS
secrets_client = boto3.client('secretsmanager', region_name='us-east-1')
s3_client = boto3.client('s3')

# Recuperación de configuración desde variables de entorno
SECRET_ARN = os.environ.get('SECRET_ARN')

def lambda_handler(event, context):
    try:
        # Recuperar las credenciales de la base de datos
        creds = get_db_credentials(SECRET_ARN)
        endpoint = creds['MySQLODBEndpoint']

        # Consultar la base de datos y obtener el resultado como un DataFrame de pandas
        df = query_database(endpoint, creds)
...
```

### **Procesamiento Basado en Eventos**  
- **AWS EventBridge** fue utilizado para **activar las funciones Lambda** cada 5 minutos, asegurando actualizaciones de datos casi en tiempo real.  
- Cada función Lambda **procesaba una tabla diferente**, asegurando modularidad.  
- Los datos procesados fueron **almacenados en un bucket S3 "Refined"**, listos para análisis.  

![alt text](image-7.png)

---

## **Seguridad y Gestión de Credenciales con AWS Secrets Manager**  

Almacenar las credenciales de base de datos en el código es riesgoso. Para mejorar la seguridad, se utilizó **AWS Secrets Manager** para almacenar de manera segura las credenciales de RDS.  

**Pasos Tomados:**  
1. Se creó un secreto para cada instancia de base de datos.  
2. Se restringió el acceso mediante **políticas IAM de AWS** (solo las funciones Lambda podían recuperar las credenciales).  
3. Se habilitó la **rotación automática de credenciales** para mejorar la seguridad.  

```json
{
  "SecretId": "rds-secret",
  "Database": "MySQL",
  "Username": "admin",
  "Password": "securepassword"
}
```

![alt text](image-8.png)


---

## **Gobernanza de Datos: Monitoreo y Registro con CloudWatch**

Hacer seguimiento de los **fallos en los trabajos de ETL** y la **salud de la base de datos** era esencial. AWS **CloudWatch Logs** ayudó a monitorear:

- **Tasas de éxito/fallo de ejecución de Lambda**
- **Tiempos de ejecución de consultas y posibles cuellos de botella**
- **Utilización de CPU y memoria de la base de datos**

### **Configuración de Alarmas en CloudWatch**
Se configuró CloudWatch para enviar **alertas por correo electrónico** si:
✅ Una función Lambda fallaba **más de 3 veces consecutivas**
✅ Una instancia RDS superaba el **80% de uso de CPU durante más de 5 minutos**

```yaml
Resources:
  LambdaErrorAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: LambdaErrorCount
      MetricName: Errors
      Namespace: AWS/Lambda
      Statistic: Sum
      Period: 60
      EvaluationPeriods: 3
      Threshold: 3
      AlarmActions:
        - !Ref SNSAlertTopic
```

![alt text](image-9.png)

---

## **Consulta de Datos y Análisis: AWS Athena y Herramientas de Visualización**

Una vez que los datos se almacenaron en el **S3 Bucket Refinado**, era necesario que fueran **fácilmente accesibles para análisis**. En lugar de configurar un almacén de datos tradicional, se utilizó **AWS Athena** para ejecutar consultas SQL **directamente sobre los datos de S3**.

### **Definición de la Tabla Athena**
```sql
CREATE EXTERNAL TABLE IF NOT EXISTS tickit_sales (
    salesid INT,
    listid INT,
    sellerid INT,
    buyerid INT,
    eventid INT,
    dateid SMALLINT,
    qtysold SMALLINT,
    pricepaid DOUBLE,
    commission DOUBLE,
    saletime STRING,
    refined_timestamp TIMESTAMP
)
STORED AS PARQUET
LOCATION 's3://utp-database-data-lake-project/tickit/refined/sales/';
```
![alt text](image-13.png)

Ahora, los datos podían ser consultados instantáneamente utilizando SQL estándar.

### **Conexión con Herramientas BI**
El paso final fue integrar Athena con **Power BI y Grafana** para visualizaciones en tiempo real.

![alt text](image-12.png)

---

## **Resumen de la Arquitectura Tecnológica**

Al final del proyecto, la arquitectura completa se veía así:

1️⃣ **AWS S3** → Almacena datos crudos y procesados.  
2️⃣ **AWS RDS (MySQL, SQL Server, PostgreSQL)** → Bases de datos que alimentan el Data Lake.  
3️⃣ **AWS CloudFormation** → Automatiza la configuración de bases de datos e infraestructura.  
4️⃣ **AWS Lambda** → Ejecuta trabajos de ETL para transformación de datos.  
5️⃣ **AWS EventBridge** → Automatiza la programación de ETL.  
6️⃣ **AWS Secrets Manager** → Gestiona credenciales de bases de datos de forma segura.  
7️⃣ **AWS CloudWatch** → Monitorea la salud del sistema y registra fallos.  
8️⃣ **AWS Athena** → Permite consultas SQL sobre datos de S3.  
9️⃣ **Power BI / Grafana** → Para informes y monitoreo.

---

# **Lecciones Aprendidas y Direcciones Futuras**

Cada proyecto trae una mezcla de éxitos y desafíos. Algunas cosas funcionan exactamente como se planeó, pero ahí es donde ocurre el mejor aprendizaje. Esta sección profundiza en mis mayores aprendizajes al construir el Data Lake basado en AWS y lo que haría diferente la próxima vez.

---

## **Lecciones Claves Aprendidas**

#### **1. Gestionar los costos en AWS requiere una planificación cuidadosa**  
Me di cuenta de que una **mala asignación de recursos** puede disparar rápidamente los costos. Mantener instancias de RDS inactivas puede generar gastos innecesarios.

✅ **Aprendizaje:**  
- Utiliza **AWS Cost Explorer** para monitorear el gasto en tiempo real.  
- Aprovecha las **políticas de ciclo de vida de S3** para mover automáticamente los datos antiguos a Glacier.  
- Considera **instancias bajo demanda vs. reservadas** para RDS si ejecutas proyectos a largo plazo.

---

#### **2. Las canalizaciones ETL deberían ser más modulares**  
Inicialmente, cada **función Lambda de AWS** manejaba un proceso de extracción de datos diferente, pero a medida que añadí más bases de datos, gestionar múltiples funciones ETL se volvió complejo. Si una función fallaba, la depuración era una pesadilla.

✅ **Aprendizaje:**  
- En lugar de múltiples funciones Lambda pequeñas, considera **orquestar flujos de trabajo ETL** con **AWS Step Functions** para un mejor manejo de errores.  
- Utiliza **Amazon Glue** en lugar de Lambda para cargas de trabajo ETL a gran escala.

---

#### **3. El registro y monitoreo son salvavidas**  
Los registros de AWS CloudWatch me ayudaron a depurar **fallos en trabajos de ETL** y **problemas de conectividad con la base de datos**.

✅ **Aprendizaje:**  
- Configura **Alarmas de CloudWatch** para recibir notificaciones de fallos.  
- Usa los **registros de consultas de Athena** para optimizar el rendimiento y evitar consultas lentas.

---

## **Mejoras Futuras y Próximos Pasos**

Este proyecto fue una excelente experiencia de aprendizaje, pero hay algunas cosas que mejoraría o expandiría en el futuro:

#### 1. **Reemplazar ETL Lambda por AWS Glue**  
AWS Glue es **sin servidor, escalable y más adecuado para tareas ETL complejas**. Pasar de Lambda a Glue reduciría la complejidad y proporcionaría una mejor gestión de esquemas.

#### 2. **Implementar permisos en el Data Lake con AWS Lake Formation**  
Actualmente, S3 almacena todos los datos, pero no hay un control de acceso detallado. **AWS Lake Formation** permitiría una mejor **gestión de permisos** mediante **políticas de acceso basadas en roles** en los conjuntos de datos almacenados.

#### 3. **Automatizar más con Terraform**  
CloudFormation fue excelente, pero **Terraform** ofrece **compatibilidad multicloud** y más flexibilidad. Migrar la automatización de la infraestructura a Terraform haría los despliegues aún más sencillos.

#### 4. **Expandir la visualización de datos con Amazon QuickSight**  
Usé **Power BI y Grafana**, pero **Amazon QuickSight** podría ser una mejor alternativa para integrarse directamente con AWS.

---

# **Conclusión**

En este proyecto, he compartido nuestro enfoque para construir un Data Lake escalable en AWS y las lecciones que aprendí en el camino. Un agradecimiento especial a mis amigos [**Andy Sanjur**](https://www.linkedin.com/in/andy-sanjur-dom%C3%ADnguez-6aa786140), [**Harris Yearwood**](https://www.linkedin.com/in/harris-yearwood/) y [**Isaac Ávila**](https://www.linkedin.com/in/isaaldair/) por sus contribuciones y apoyo durante todo este proceso.

A lo largo del camino, enfrentamos varios **desafíos** depurando problemas de red, optimizando la eficiencia de costos y mejorando las medidas de seguridad. Pero al superarlos, adquirimos **valiosa experiencia práctica** con los servicios de AWS y la infraestructura como código (IaC).

---

## **Reflexiones Finales y Lecciones Aprendidas**

✅ **La infraestructura en la nube requiere un equilibrio entre automatización y flexibilidad**  
El uso de CloudFormation simplificó el despliegue, pero ajustar las configuraciones requirió intervención manual. La próxima vez, exploraría **Terraform** para más flexibilidad.

✅ **Las canalizaciones ETL deben ser escalables y mantenibles**  
Lambda funcionó para ETL a pequeña escala, pero **AWS Glue** sería una mejor solución a largo plazo. **Step Functions** también podría mejorar el manejo de errores.

✅ **La optimización de costos es un proceso continuo**  
Incluso dentro del nivel gratuito, los costos de AWS pueden dispararse si no se monitorean. Herramientas como **Cost Explorer** y **políticas de ciclo de vida de S3** ayudan a controlar los gastos.

✅ **La seguridad debe ser una prioridad desde el principio**  
Almacenar credenciales en **AWS Secrets Manager**, implementar permisos basados en roles de **IAM** y bloquear el acceso público a S3 fueron medidas clave de seguridad.

---

Thank you for reading! If you have any questions or comments, please feel free to contact me. Your feedback is highly appreciated.

---

**Keywords:** AWS, Data Lake, Cloud Computing, Database Systems, ETL, CloudFormation, S3, RDS, Lambda, Data Engineering  

---