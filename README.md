# DataProject
![Logotipo](https://n3m5z7t4.rocketcdn.me/wp-content/plugins/edem-shortcodes/public/img/logo-Edem.png)
# Arquitectura en tiempo real sin servidor de IoT
Procesamiento de datos sin servidor | EDEM 2022

#### Descripción del caso
Wake es una startup enfocada en la arquitectura sostenible. Uno de sus muchos retos es la reducción del consumo eléctrico en los edificios. Para lograr este desafío, han lanzado una RFP para monitorear la temperatura y la humedad con el fin de regular la temperatura óptima de nuestros hogares.

#### Desafíos empresariales

- Parte 01: Monitorear la temperatura y humedad registradas y mostrarlas en un tablero para ayudar a las partes interesadas a tomar decisiones.
- Parte 02: Regular la climatización cuando se registra un rango de temperatura inadecuado durante un determinado periodo.

#### Arquitectura de datos
<img src="00_DocAux/iot_arch.jpg" width="700"/>

# Parte 01: Procesamiento de datos sin servidor con Dataflow

## Requisitos de configuración
- [Google Cloud Platform - Prueba gratuita](https://console.cloud.google.com/freetrial)
- Clona este **repositorio**
- Para esta demostración, trabajaremos en un **Cloud Shell**.
- Habilite las *Google Cloud API* requeridas ejecutando los siguientes comandos:

```
Los servicios de gcloud habilitan dataflow.googleapis.com
Los servicios de gcloud habilitan cloudiot.googleapis.com
Los servicios de gcloud habilitan cloudbuild.googleapis.com
```
- Crear entorno Python
```
virtualenv -p python3 <NOMBRE_ENTORNO>
fuente <NOMBRE_AMBIENTE>/bin/activate
```
- Instale las dependencias de python ejecutando el siguiente comando:

```
pip install -U -r setup_dependencies.txt
```

## PubSub
En primer lugar, crearemos dos **Temas** y sus **Suscripciones** predeterminadas.

- Vaya a la página de Cloud Console [PubSub](https://console.cloud.google.com/cloudpubsub).
- Haga clic en **Crear tema**, proporcione un nombre de tema único y marque la opción **agregar suscripción predeterminada**.

Tanto los temas como las suscripciones son necesarios en los siguientes pasos para crear la canalización de datos.

## Almacenamiento en la nube

Vaya a la página de Cloud Console [Cloud Storage](https://console.cloud.google.com/storage).

- Cree un **depósito** especificando un nombre único global. Este depósito se usará para almacenar la plantilla de Dataflow Flex.

## Núcleo de IoT

Para esta demostración, usaremos Cloud Shell como un simulador de datos de IoT.

- Vaya a la página de Cloud Console [IoT Core](https://console.cloud.google.com/iot).
- Cree un registro de IoT eligiendo uno de los temas de PubSub creados anteriormente.
- Vaya a Cloud Shell y genere una **clave RSA con certificado X509 autofirmado**. más [información](https://cloud.google.com/iot/docs/how-tos/credentials/keys#generating_an_rsa_key)
- Una vez que haya creado tanto el registro como la clave RSA, registre un dispositivo y cargue el archivo *rsa_cert.pem* en la sección *autenticación*.

Ahora, hemos vinculado nuestro dispositivo (Cloud Shell) con IoT Core.

<img src="00_DocAux/iot_ui.PNG" ancho="700"/>

## BigQuery

- Vaya a la página de Cloud Console [BigQuery](https://console.cloud.google.com/bigquery).
- Cree un **Conjunto de datos de BigQuery** especificando UE como ubicación de datos.
- No se requerirá nada más, ya que Dataflow Pipeline creará la tabla de BigQuery.

## Flujo de datos

- Vaya a [Carpeta de flujo de datos] (https://github.com/jabrio/Serverless_EDEM/tree/main/02_Dataflow) y siga las instrucciones ubicadas en el archivo **edemDataflow.py** para procesar los datos mediante la canalización de Beam.
- En esta demostración, crearemos una **Plantilla Dataflow Flex**. Más [información](https://cloud.google.com/dataflow/docs/guides/templates/using-flex-templates).
- Tienes los archivos necesarios en la carpeta Dataflow (*Dockerfile* y *requirements.txt*).
- [Empaquete su código de Python en una imagen de Docker](https://cloud.google.com/dataflow/docs/guides/templates/using-flex-templates#python_only_creating_and_building_a_container_image) y guárdelo en Container Registry. Puede hacer esto ejecutando el siguiente comando:

```
gcloud compilaciones enviar --tag 'gcr.io/<YOUR_PROJECT_ID>/<YOUR_FOLDER_NAME>/<YOUR_IMAGE_NAME>:latest' .
```
- [Crear plantilla de Dataflow Flex](https://cloud.google.com/dataflow/docs/guides/templates/using-flex-templates#creating_a_flex_template) desde la imagen de Docker:

```
gcloud dataflow flex-template build "gs://<SU_BUCKET_NAME>/<SU_TEMPLATE_NAME>.json" \
  --image "gcr.io/<ID_DE_TU_PROYECTO>/<NOMBRE_DE_TU_CARPETA>/<NOMBRE_DE_TU_IMAGEN>:último" \
  --sdk-lenguaje "PYTHON"
```

- Finalmente, ejecute un [trabajo de Dataflow desde plantilla](https://cloud.google.com/dataflow/docs/guides/templates/using-flex-templates#running_a_flex_template_pipeline):

```
gcloud dataflow flex-template ejecuta "<TU_FLUJO_DE_DATOS_NOMBRE_TRABAJO>" \
    --template-file-gcs-location "gs://<SU_BUCKET_NAME>/<SU_TEMPLATE_NAME>.json" \
    --región "europa-oeste1"
```

<img src="00_DocAux/dataflow_ui.PNG" ancho="700"/>

## Enviar datos desde el dispositivo

- Vaya a [carpeta IoTCore](https://github.com/jabrio/Serverless_EDEM/tree/main/01_IoTCore) y ejecute el siguiente comando para comenzar a generar datos.

```
python edemDeviceData.py \
    --algoritmo RS256 \
    --cloud_region europa-oeste1 \
    --device_id <SU_NOMBRE_DISPOSITIVO_IOT> \
    --private_key_file rsa_private.pem \
    --project_id <TU_PROYECTO_ID> \
    --registry_id <SU_REGISTRO_IOT>
```

## Verifique que los datos estén llegando y visualícelos con Data Studio

- Vaya a [BigQueryUI](https://console.cloud.google.com/bigquery) y debería ver su tabla de bigquery ya creada.

<img src="00_DocAux/bq_ui.PNG" ancho="700"/>

- Vaya a [**Data Studio**](https://datastudio.google.com/). Vincula tu tabla de BigQuery.
- Cree un Tablero como se muestra a continuación, que represente la temperatura y la humedad del dispositivo.
<img src="00_DocAux/Tablero.PNG" width="700"/>

# Parte 02: Arquitectura basada en eventos con Cloud Functions

- Vaya a [carpeta CloudFunctions]() y siga las instrucciones ubicadas en el archivo edemCloudFunctions.py.
- Vaya a la página de Cloud Console [Cloud Functions] (https://console.cloud.google.com/functions).
- Haga clic en **Crear función** (europe-west1) y seleccione **PubSub** como tipo de activador y haga clic en **guardar**.
- Haga clic en **Siguiente** y elija **Python 3.9** como tiempo de ejecución.
- Copie su código en el archivo Main.py y las dependencias de python en requirements.txt.
- cuando haya terminado, haga clic en **implementar**.
- Si una temperatura agregada por minuto está fuera de rango, **se lanzará un comando al dispositivo y su configuración se actualizará**. Puede verificar eso yendo a la pestaña *configuración y estado* en la página del dispositivo IoT.
- Información útil: [ejemplos de código de IoT Core](https://cloud.google.com/iot/docs/samples)

# Solución
- [Arquitectura sin servidor en tiempo real de IoT Parte 01](https://www.youtube.com/watch?v=gXngs3pTYJ8)
- [Arquitectura sin servidor en tiempo real de IoT Parte 02](https://www.youtube.com/watch?v=mh8kNW1OOAU)
