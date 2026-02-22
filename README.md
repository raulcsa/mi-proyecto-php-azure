# Pipeline CI/CD con Jenkins y Azure para Proyecto PHP

Este proyecto demuestra cómo montar una pipeline de Integración Continua y Despliegue Continuo (CI/CD) automatizada. Cada vez que se realiza un cambio en este repositorio, Jenkins descarga el código, lo prepara y lo despliega automáticamente en un **Azure App Service**.

## Preparativos

Para replicar este entorno, necesitas lo siguiente:
1. **Jenkins** instalado y funcionando (local, en máquina virtual o Docker).
2. **Cuenta de Microsoft Azure** con una suscripción activa.
3. **Cuenta de GitHub** (o cualquier repositorio Git).

---

##  Paso a Paso

### Paso 1: Crear el proyecto PHP
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

## 2. Crear un Service Principal (Mis credenciales para Jenkins)

Para que Jenkins tenga permiso de "hablar" con mi cuenta de Azure, voy a generar un Service Principal. Abro la **Azure Cloud Shell**  y ejecuto:

az ad sp create-for-rbac --name "JenkinsCI" --role contributor --scopes /subscriptions/<Mi_ID_DE_SUSCRIPCION>
<img width="1793" height="150" alt="image" src="https://github.com/user-attachments/assets/05c921e0-a678-48a2-a005-503ecaae5065" />
<img width="1919" height="817" alt="image" src="https://github.com/user-attachments/assets/765fc116-db36-4b26-9c60-def6546840ed" />
<img width="1334" height="688" alt="image" src="https://github.com/user-attachments/assets/48481c99-e06a-4099-b6e2-39ad0dc517f1" />
<img width="1919" height="604" alt="image" src="https://github.com/user-attachments/assets/066466c7-351e-4e83-ba5f-88edfbc29300" />
<img width="1919" height="342" alt="image" src="https://github.com/user-attachments/assets/0d9495d1-f515-4c7a-941e-0e6d4395d5ce" />
<img width="852" height="274" alt="image" src="https://github.com/user-attachments/assets/b6947be6-8545-4d09-b1e8-163951b7be98" />
<img width="980" height="301" alt="image" src="https://github.com/user-attachments/assets/4f52d178-423e-4d1b-b03a-607b1f457c08" />
<img width="807" height="797" alt="image" src="https://github.com/user-attachments/assets/6a929f2c-626f-4d99-b8f5-3c1834ed927c" />












