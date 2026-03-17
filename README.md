# Pipeline CI/CD con Jenkins y Azure para Proyecto en PHP

![PHP](https://img.shields.io/badge/PHP-8.2-777BB4?logo=php&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-App%20Service-0078D4?logo=microsoftazure&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

Este proyecto demuestra cómo montar una pipeline de **Integración Continua y Despliegue Continuo (CI/CD)** completamente automatizada. Cada vez que se realiza un `git push` a este repositorio, Jenkins descarga el código, lo empaqueta y lo despliega automáticamente en un **Azure App Service**.

---

## Tabla de contenidos

1. [Tecnologías utilizadas](#tecnologías-utilizadas)
2. [Cómo funciona](#cómo-funciona)
3. [Estructura del proyecto](#estructura-del-proyecto)
4. [Prerrequisitos](#prerrequisitos)
5. [Paso a paso](#paso-a-paso)
   - [1. Crear el proyecto PHP](#1-crear-el-proyecto-php)
   - [2. Crear el Azure App Service](#2-crear-el-azure-app-service)
   - [3. Crear un Service Principal](#3-crear-un-service-principal-credenciales-para-jenkins)
   - [4. Crear el pipeline en Jenkins](#4-crear-el-pipeline-en-jenkins)
   - [5. Configurar las credenciales de Azure en Jenkins](#5-configurar-las-credenciales-de-azure-en-jenkins)
   - [6. Configurar el Webhook en GitHub](#6-configurar-el-webhook-en-github)
   - [7. Verificar el despliegue](#7-verificar-el-despliegue)

---

## Tecnologías utilizadas

| Tecnología | Propósito |
|---|---|
| **PHP 8.2** | Lenguaje de la aplicación web |
| **Azure App Service** | Plataforma de alojamiento en la nube |
| **Jenkins** | Servidor de automatización CI/CD |
| **Azure CLI** | Herramienta para desplegar desde la línea de comandos |
| **GitHub Webhooks** | Disparador automático del pipeline |
| **ngrok** | Túnel para exponer Jenkins localmente |

---

## Cómo funciona

El flujo de trabajo automatizado es el siguiente:

```
Developer → git push → GitHub → Webhook → Jenkins → Azure App Service
```

1. El desarrollador hace un `git push` al repositorio de GitHub.
2. GitHub envía una notificación HTTP (webhook) al servidor Jenkins.
3. Jenkins clona el repositorio, empaqueta el código en un archivo `.zip` y ejecuta el `Jenkinsfile`.
4. El `Jenkinsfile` usa la **Azure CLI** para autenticarse con un Service Principal y despliega el `.zip` en Azure App Service.
5. La aplicación queda disponible en `https://<APP_NAME>.azurewebsites.net`.

---

## Estructura del proyecto

```
mi-proyecto-php-azure-jenkins/
├── index.php        # Aplicación PHP principal
├── Jenkinsfile      # Definición declarativa del pipeline CI/CD
└── README.md        # Documentación del proyecto
```


## Prerrequisitos

Antes de replicar este entorno, necesitas:

- **Jenkins** instalado y funcionando (local, en máquina virtual o contenedor Docker).
  - Plugin requerido: [Azure Credentials Plugin](https://plugins.jenkins.io/azure-credentials/)
  - Plugin requerido: [GitHub Integration Plugin](https://plugins.jenkins.io/github/)
- **Azure CLI** instalada en el agente de Jenkins (`az` disponible en el PATH).
- **Cuenta de Microsoft Azure** con una suscripción activa.
- **Cuenta de GitHub** con acceso de administrador al repositorio (para configurar webhooks).
- **ngrok** (u otro túnel inverso) si Jenkins no tiene una IP pública accesible desde Internet.

---

## Paso a paso

### 1. Crear el proyecto PHP

Creamos un archivo base `index.php` en la raíz del repositorio. Este será el código que desplegaremos en Azure:

```php
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Pipeline Jenkins + Azure</title>
</head>
<body>
    <h1>¡Hola Mundo! Despliegue automatizado con Jenkins y Azure</h1>
    <p>Fecha y hora actual: <?php echo date('Y-m-d H:i:s'); ?></p>
    <p>Versión de PHP: <?php echo phpversion(); ?></p>
</body>
</html>
```

<img width="754" height="243" alt="index.php en el repositorio" src="https://github.com/user-attachments/assets/74b099fe-4be7-4a58-8677-93e80afaec53" />

---

### 2. Crear el Azure App Service

En el **Portal de Azure**, creamos el servicio donde vivirá nuestra aplicación:

1. Busca **"App Services"** y haz clic en **Crear → Aplicación web**.
2. Selecciona tu **Suscripción** y crea un nuevo **Grupo de recursos** (p. ej. `rg-jenkins-demo`).
3. Asigna un **Nombre único** a la app (p. ej. `mi-app-php-12345`).
4. En **Publicar**, selecciona **Código**.
5. En **Pila del entorno en tiempo de ejecución**, elige **PHP 8.2**.
6. Como **Sistema operativo**, selecciona **Linux**.
7. Haz clic en **Revisar y crear** → **Crear**.

<img width="715" height="728" alt="Creación del App Service en Azure" src="https://github.com/user-attachments/assets/ac3dc7d9-b7fe-41fb-bc5e-fbfa66bbc60c" />
<img width="759" height="398" alt="Configuración del App Service" src="https://github.com/user-attachments/assets/18aed775-7a8f-4296-a175-e65b5f7167ad" />

---

### 3. Crear un Service Principal (credenciales para Jenkins)

Para que Jenkins pueda autenticarse con Azure, necesitamos un **Service Principal** con rol de *Contributor*. Abre la **Azure Cloud Shell** y ejecuta:

```bash
az ad sp create-for-rbac \
  --name "JenkinsCI" \
  --role contributor \
  --scopes /subscriptions/<TU_ID_DE_SUSCRIPCION>
```

El comando devuelve un JSON con los campos `appId`, `password`, `tenant` e `id`. **Guárdalos en un lugar seguro**, los necesitarás en el paso 5.

<img width="1793" height="150" alt="Salida del comando az ad sp create-for-rbac" src="https://github.com/user-attachments/assets/05c921e0-a678-48a2-a005-503ecaae5065" />

> ⚠️ **Seguridad:** Nunca guardes estas credenciales en el código fuente. Utiliza siempre el almacén de credenciales de Jenkins.

---

### 4. Crear el pipeline en Jenkins

1. En Jenkins, haz clic en **Nueva tarea** → selecciona **Pipeline**.
2. En la sección **Pipeline**, elige **Pipeline script from SCM**.
3. En **SCM**, selecciona **Git** e introduce la URL de tu repositorio:
   ```
   https://github.com/raulcsa/mi-proyecto-php-azure-jenkins.git
   ```
4. En **Script Path**, deja el valor por defecto: `Jenkinsfile`.
5. En **Build Triggers**, activa **GitHub hook trigger for GITScm polling** para que el pipeline se ejecute automáticamente con cada `push`.

<img width="1919" height="817" alt="Configuración del pipeline en Jenkins" src="https://github.com/user-attachments/assets/765fc116-db36-4b26-9c60-def6546840ed" />
<img width="1334" height="688" alt="Configuración de SCM en Jenkins" src="https://github.com/user-attachments/assets/48481c99-e06a-4099-b6e2-39ad0dc517f1" />
<img width="852" height="274" alt="Build Triggers en Jenkins" src="https://github.com/user-attachments/assets/b6947be6-8545-4d09-b1e8-163951b7be98" />

---

### 5. Configurar las credenciales de Azure en Jenkins

Ve a **Manage Jenkins → Credentials → Global → Add Credentials** y selecciona el tipo **Azure Service Principal**. Rellena los campos con los datos del JSON obtenido en el paso 3:

| Campo | Valor |
|---|---|
| **Subscription ID** | ID de tu suscripción de Azure |
| **Client ID** | Valor `appId` del JSON |
| **Client Secret** | Valor `password` del JSON |
| **Tenant ID** | Valor `tenant` del JSON |
| **Azure Environment** | `Azure` (nube pública global) |
| **ID** | `azure-credentials` ← este nombre debe coincidir con `AZURE_CREDENTIALS_ID` en el `Jenkinsfile` |

<img width="1912" height="819" alt="Pantalla de credenciales globales en Jenkins" src="https://github.com/user-attachments/assets/cb1eabd4-fe2d-4fbc-92b5-340e5809ecc6" />
<img width="546" height="756" alt="Formulario Add Azure Service Principal" src="https://github.com/user-attachments/assets/bce3f7cc-cbbd-40ae-8a2b-89a8dfe6cf26" />
<img width="542" height="338" alt="Credencial creada correctamente" src="https://github.com/user-attachments/assets/dabe4d24-38a1-43d1-adc3-4a78d8fad5be" />

---

### 6. Configurar el Webhook en GitHub

Para que GitHub notifique a Jenkins automáticamente en cada `push`, ve a **Settings → Webhooks → Add webhook** en tu repositorio y configura:

| Campo | Valor |
|---|---|
| **Payload URL** | `https://<tu-url-ngrok>/github-webhook/` |
| **Content type** | `application/json` |
| **Events** | Solo **"Just the push event"** |
| **Active** | ✅ Activado |

> 💡 Si Jenkins está en tu máquina local (sin IP pública), usa [ngrok](https://ngrok.com/) para generar una URL pública temporal: `ngrok http 8080`.

<img width="807" height="797" alt="Configuración del Webhook en GitHub" src="https://github.com/user-attachments/assets/6a929f2c-626f-4d99-b8f5-3c1834ed927c" />

---

### 7. Verificar el despliegue

Haz un `git push` al repositorio. Jenkins debería:

1. Detectar el evento vía webhook.
2. Ejecutar el pipeline (clonar → empaquetar → desplegar).
3. Mostrar el resultado en la vista de **Stage View**.

Una vez completado con éxito, la aplicación estará disponible en:

```
https://<APP_NAME>.azurewebsites.net
```

<img width="1918" height="841" alt="Pipeline ejecutado con éxito en Jenkins" src="https://github.com/user-attachments/assets/c595c6df-11a0-45c3-8c7f-6904449f5bc2" />
<img width="1287" height="331" alt="Aplicación PHP desplegada en Azure" src="https://github.com/user-attachments/assets/0c211f8f-7f4c-4d91-a61f-d23038603ee1" />

---

## Licencia

Este proyecto se distribuye bajo la licencia [MIT](https://opensource.org/licenses/MIT). Siéntete libre de usarlo, modificarlo y compartirlo.
