# DataProject
prueba data project 2
Logo
Arquitectura en tiempo real sin servidor IoT
Procesamiento de datos sin servidor | EDEM 2022

Descripción del caso
Wake es una startup enfocada en la arquitectura sostenible. Uno de sus muchos retos es la reducción del consumo eléctrico en los edificios. Para lograr este desafío, han lanzado una RFP para monitorear la temperatura y la humedad con el fin de regular la temperatura óptima de nuestros hogares.

Desafíos empresariales
Parte 01: monitorear la temperatura y la humedad registradas y mostrarlas en un tablero para ayudar a las partes interesadas a tomar decisiones.
Parte 02: Regular la climatización cuando se registra un rango de temperatura inadecuado durante un período determinado.
Arquitectura de datos


Parte 01: Procesamiento de datos sin servidor con Dataflow
Requisitos de configuración
Google Cloud Platform - Prueba gratuita
Clonar este repositorio
Para esta demostración, trabajaremos en un Cloud Shell .
Habilite las API de Google Cloud requeridas ejecutando los siguientes comandos:
gcloud services enable dataflow.googleapis.com
gcloud services enable cloudiot.googleapis.com
gcloud services enable cloudbuild.googleapis.com
Crear entorno Python
virtualenv -p python3 <ENVIRONTMENT_NAME>
source <ENVIRONMENT_NAME>/bin/activate
Instale las dependencias de Python ejecutando el siguiente comando:
pip install -U -r setup_dependencies.txt
PubSub
En primer lugar, crearemos dos Temas y sus Suscripciones predeterminadas .

Vaya a la página PubSub de Cloud Console .
Haga clic en Crear tema , proporcione un nombre de tema único y marque la opción Agregar suscripción predeterminada .
Tanto los temas como las suscripciones son necesarios en los siguientes pasos para crear la canalización de datos.

Almacenamiento en la nube
Vaya a la página de almacenamiento en la nube de Cloud Console .

Cree un depósito especificando un nombre único global. Este depósito se usará para almacenar la plantilla de Dataflow Flex.
Núcleo de IoT
Para esta demostración, usaremos Cloud Shell como un simulador de datos de IoT.

Vaya a la página de Cloud Console IoT Core .
Cree un registro de IoT eligiendo uno de los temas de PubSub creados anteriormente.
Vaya a Cloud Shell y genere una clave RSA con certificado X509 autofirmado . más información
Una vez que haya creado tanto el registro como la clave RSA, registre un dispositivo y cargue el archivo rsa_cert.pem en la sección de autenticación .
Ahora, hemos vinculado nuestro dispositivo (Cloud Shell) con IoT Core.



BigQuery
Vaya a la página de BigQuery de Cloud Console .
Cree un conjunto de datos de BigQuery especificando EU como ubicación de datos.
No se requerirá nada más, ya que Dataflow Pipeline creará la tabla de BigQuery.
Flujo de datos
Vaya a la carpeta Dataflow y siga las instrucciones ubicadas en el archivo edemDataflow.py para procesar los datos mediante la canalización de Beam.
En esta demostración, crearemos una plantilla de Dataflow Flex . Más información _
Tiene los archivos necesarios en la carpeta Dataflow ( Dockerfile y requirements.txt ).
Empaqueta tu código python en una imagen de Docker y guárdalo en Container Registry. Puede hacer esto ejecutando el siguiente comando:
gcloud builds submit --tag 'gcr.io/<YOUR_PROJECT_ID>/<YOUR_FOLDER_NAME>/<YOUR_IMAGE_NAME>:latest' .
Cree una plantilla de Dataflow Flex a partir de una imagen de Docker:
gcloud dataflow flex-template build "gs://<YOUR_BUCKET_NAME>/<YOUR_TEMPLATE_NAME>.json" \
  --image "gcr.io/<YOUR_PROJECT_ID>/<YOUR_FOLDER_NAME>/<YOUR_IMAGE_NAME>:latest" \
  --sdk-language "PYTHON" 
Finalmente, ejecute un trabajo de Dataflow desde la plantilla :
gcloud dataflow flex-template run "<YOUR_DATAFLOW_JOB_NAME>" \
    --template-file-gcs-location "gs://<YOUR_BUCKET_NAME>/<YOUR_TEMPLATE_NAME>.json" \
    --region "europe-west1"


Enviar datos desde el dispositivo
Vaya a la carpeta IoTCore y ejecute el siguiente comando para comenzar a generar datos.
python edemDeviceData.py \
    --algorithm RS256 \
    --cloud_region europe-west1 \
    --device_id <YOUR_IOT_DEVICE_NAME> \
    --private_key_file rsa_private.pem \
    --project_id <YOUR_PROJECT_ID> \
    --registry_id <YOUR_IOT_REGISTRY>
Verifique que los datos estén llegando y visualícelos con Data Studio
Vaya a BigQueryUI y debería ver su tabla de bigquery ya creada.


Vaya a Estudio de datos . Vincula tu tabla de BigQuery.
Cree un Tablero como se muestra a continuación, que represente la temperatura y la humedad del dispositivo.


Parte 02: Arquitectura basada en eventos con Cloud Functions
Vaya a la carpeta CloudFunctions y siga las instrucciones ubicadas en el archivo edemCloudFunctions.py.
Vaya a la página Funciones en la nube de Cloud Console .
Haga clic en Crear función (europe-west1) y elija PubSub como tipo de activador y haga clic en guardar .
Haga clic en Siguiente y elija Python 3.9 como tiempo de ejecución.
Copie su código en el archivo Main.py y las dependencias de python en requirements.txt.
cuando haya terminado, haga clic en implementar .
Si una temperatura agregada por minuto está fuera de rango, se lanzará un comando al dispositivo y se actualizará su configuración . Puede verificar eso yendo a la pestaña de configuración y estado en la página del dispositivo IoT.
Información útil: Ejemplos de código de IoT Core
Solución
IoT Arquitectura sin servidor en tiempo real Parte 01
IoT Arquitectura sin servidor en tiempo real Parte 02
