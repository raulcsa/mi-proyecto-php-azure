# Pipeline CI/CD con Jenkins y Azure para Proyecto en PHP 

Este proyecto demuestra cómo montar una pipeline de Integración Continua y Despliegue Continuo (CI/CD) automatizada. Cada vez que se realiza un cambio en este repositorio, Jenkins descarga el código, lo prepara y lo despliega automáticamente en un **Azure App Service**.

## Preparativos

Para replicar este entorno, necesitas lo siguiente:
1. **Jenkins** instalado y funcionando (local, en máquina virtual o Docker).
2. **Cuenta de Microsoft Azure** con una suscripción activa.
3. **Cuenta de GitHub** (o cualquier repositorio Git).

---

##  Paso a Paso

## 1. Crear el proyecto PHP
Creamos un archivo base `index.php` en la raíz del repositorio. Este será el código que desplegaremos.
<img width="754" height="243" alt="image" src="https://github.com/user-attachments/assets/74b099fe-4be7-4a58-8677-93e80afaec53" />



## 2. Crear mi Azure App Service

Primero, necesito preparar el entorno donde vivirá mi aplicación:

1.  Entro al **Portal de Azure** y busco "App Services".
2.  Hago clic en **Crear** -> **Aplicación web**.
3.  Selecciono mi **Suscripción** y creo un nuevo **Grupo de recursos** (por ejemplo: `rg-jenkins-demo`).
4.  Le asigno un **Nombre único** a la app (por ejemplo: `mi-app-php-12345`).
5.  En la sección de **Publicar**, marco la opción **Código**.
6.  En **Pila del entorno en tiempo de ejecución**, elijo **PHP 8.2**.
7.  Como **Sistema operativo**, selecciono **Linux**.
8.  Hago clic en **Revisar y crear** y, tras validar, pulso **Crear**.

<img width="715" height="728" alt="image" src="https://github.com/user-attachments/assets/ac3dc7d9-b7fe-41fb-bc5e-fbfa66bbc60c" />
<img width="759" height="398" alt="image" src="https://github.com/user-attachments/assets/18aed775-7a8f-4296-a175-e65b5f7167ad" />

---

## 3. Crear un Service Principal (Mis credenciales para Jenkins)

Para que Jenkins tenga permiso de "hablar" con mi cuenta de Azure, voy a generar un Service Principal. Abro la **Azure Cloud Shell**  y ejecuto:

```az ad sp create-for-rbac --name "JenkinsCI" --role contributor --scopes /subscriptions/<Mi_ID_DE_SUSCRIPCION>```
<img width="1793" height="150" alt="image" src="https://github.com/user-attachments/assets/05c921e0-a678-48a2-a005-503ecaae5065" />

---
## 4. Crear el pipeline para el deslpiegue
<img width="1919" height="817" alt="image" src="https://github.com/user-attachments/assets/765fc116-db36-4b26-9c60-def6546840ed" />
<img width="1334" height="688" alt="image" src="https://github.com/user-attachments/assets/48481c99-e06a-4099-b6e2-39ad0dc517f1" />

Esta es la parte donde le digo a Jenkins de dónde tiene que sacar el código para ejecutar el pipeline. 

**Repository URL:** `https://github.com/raulcsa/mi-proyecto-php-azure.git`. Esta es la dirección pública de mi repositorio en GitHub. Jenkins usará esta URL para hacer el `git clone` y descargarse mi proyecto y el archivo `Jenkinsfile`.

<img width="852" height="274" alt="image" src="https://github.com/user-attachments/assets/b6947be6-8545-4d09-b1e8-163951b7be98" />

Esta área sirve para definir qué eventos automáticos deben hacer que mi pipeline comience a ejecutarse por sí sola, sin que nadie tenga que pulsar el botón de "Construir ahora".

---
## 5. Configurar las credenciales para conectar con Azure
<img width="1912" height="819" alt="Captura de pantalla 2026-02-22 194038" src="https://github.com/user-attachments/assets/cb1eabd4-fe2d-4fbc-92b5-340e5809ecc6" />
<img width="546" height="756" alt="Captura de pantalla 2026-02-22 194344" src="https://github.com/user-attachments/assets/bce3f7cc-cbbd-40ae-8a2b-89a8dfe6cf26" />
<img width="542" height="338" alt="Captura de pantalla 2026-02-22 194401" src="https://github.com/user-attachments/assets/dabe4d24-38a1-43d1-adc3-4a78d8fad5be" />

En estas imágenes estoy en la pantalla "Add Azure Service Principal" dentro de la sección de credenciales globales de Jenkins. Aquí es exactamente donde estoy vinculando a Jenkins con mi cuenta de Azure, utilizando los datos del JSON que obtuve en la terminal durante el Paso 2.
* **Subscription ID:** Aquí he puesto el ID de mi suscripción de Azure (`e24f79c4-03aa...`). Básicamente le estoy diciendo a Jenkins bajo qué cuenta de facturación y proyecto de Azure voy a trabajar.
* **Client ID:** Aquí he pegado el valor `appId` de mi JSON (`61056f0a-5ec1...`). Este es el "nombre de usuario" público del bot (Service Principal) que creé para Jenkins.
* **Client Secret:** En este campo he pegado el `password` que me devolvió el JSON. Como es una contraseña real. Esta es la "llave maestra" de mi bot.
* **Tenant ID:** Aquí he introducido el identificador de mi directorio de Azure Active Directory (`19544f2f-ebf4...`). 
* **Azure Environment:** Lo he dejado por defecto en `Azure` porque estoy utilizando la nube pública global estándar de Microsoft (no la de China ni la gubernamental de EE. UU.).
* **ID:** Le he puesto el nombre `azure-credentials`. **Este es el campo más importante para mí**. Es el apodo interno que Jenkins le dará a este conjunto de credenciales. (`AZURE_CREDENTIALS_ID = 'azure-credentials'`).

---
## 6. Configurar webhook
<img width="807" height="797" alt="image" src="https://github.com/user-attachments/assets/6a929f2c-626f-4d99-b8f5-3c1834ed927c" />

En esta captura muestro cómo he configurado el Webhook en mi repositorio para disparar las ejecuciones de mi pipeline CI/CD en Jenkins.

* **Payload URL:** Aquí he configurado el *endpoint* de destino. He introducido la URL pública generada por ngrok (`https://nonhereditably-insightful-journey.ngrok-free.dev/github-webhook/`). 
* **Content type:** Lo he seteado explícitamente a `application/json`. Esto es vital porque el *listener* del plugin en Jenkins espera parsear el cuerpo de la petición HTTP como un objeto JSON estructurado que contiene los metadatos del evento, en lugar de utilizar el formato `x-www-form-urlencoded`.
* **Events:** He limitado el alcance del *trigger* seleccionando la opción **"Just the push event"**. Con esto optimizo el uso de recursos de mi infraestructura, garantizando que el pipeline solo se ponga en marcha cuando se haga un `git push` con nuevos *commits* al repositorio, ignorando el resto de eventos de la API de GitHub (como creación de *Pull Requests*, etiquetas o comentarios).
* **Active:** He dejado este *flag* activado para habilitar el *delivery* de los *webhooks* en tiempo real desde este mismo momento.

---
## 7. Probar que todo funciona y se sube el codigo al app service
<img width="1918" height="841" alt="image" src="https://github.com/user-attachments/assets/c595c6df-11a0-45c3-8c7f-6904449f5bc2" />

<img width="1287" height="331" alt="image" src="https://github.com/user-attachments/assets/0c211f8f-7f4c-4d91-a61f-d23038603ee1" />














