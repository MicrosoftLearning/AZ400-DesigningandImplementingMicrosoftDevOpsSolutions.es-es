---
lab:
  title: Implementación de contenedores de Docker en aplicaciones web de Azure App Service
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Implementación de contenedores de Docker en aplicaciones web de Azure App Service

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique una suscripción de Azure existente o cree una.

- Compruebe que tiene una cuenta de Microsoft o de Microsoft Entra con el rol Propietario o Colaborador en la suscripción a Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Introducción al laboratorio

En este laboratorio, aprenderá a usar una canalización de CI/CD de Azure DevOps para compilar una imagen de Docker personalizada, insertarla en Azure Container Registry e implementarla como un contenedor en Azure App Service.

## Objetivos

Después de completar este laboratorio, podrá:

- Crear una imagen de Docker personalizada mediante un agente de Linux hospedado por Microsoft.
- Insertar una imagen en Azure Container Registry.
- Implementar una imagen de Docker como un contenedor en Azure App Service mediante Azure DevOps.

## Tiempo estimado: 30 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y elige **Scrum** en la lista desplegable **Proceso del elemento de trabajo**. Haga clic en **Crear**.

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

1. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - Definiciones de flujo de trabajo de GitHub del contenedor de carpetas **.github**.
    - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

#### Tarea 3: (omitir si ya la has completado) Establecer la rama principal como rama predeterminada

1. Ve a **Repos>Ramas**.
1. Mantén el puntero sobre la rama **main** y haz clic en los puntos suspensivos a la derecha de la columna.
1. Haz clic en **Establecer como rama predeterminada**.

### Ejercicio 1: Administrar la conexión de servicio

En este ejercicio, configurarás la conexión de servicio con la suscripción de Azure y luego importarás y ejecutarás la canalización de CI.

#### Tarea 1: (omite si ya lo has hecho) Administrar la conexión de servicio

Puede crear una conexión desde Azure Pipelines a servicios externos y remotos para ejecutar las tareas de un trabajo.

En esta tarea, crearás una entidad de servicio mediante la CLI de Azure, que permitirá a Azure DevOps realizar lo siguiente:

- Implementar recursos en tu suscripción de Azure.
- Inserte la imagen de Docker en Azure Container Registry.
- Agregar una asignación de roles para permitir que App de Azure Service extraiga la imagen de Docker de Azure Container Registry.

> **Nota**: si ya tienes una entidad de servicio, puedes continuar directamente con la próxima tarea.

Necesitarás una entidad de servicio para implementar recursos de Azure desde Azure Pipelines.

Azure Pipeline crea automáticamente una entidad de servicio cuando se conecta a una suscripción de Azure desde dentro de una definición de canalización o al crear una nueva conexión de servicio desde la página de configuración del proyecto (opción automática). También puedes crear manualmente la entidad de servicio desde el portal o mediante la CLI de Azure, y volver a usarla en otros proyectos.

1. En el equipo del laboratorio, abre un explorador web, ve al [**Portal de Azure**](https://portal.azure.com) e inicia sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que vas a usar en este laboratorio, así como el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción.
1. En el portal de Azure portal, haz clic en el icono de **Cloud Shell**, situado inmediatamente a la derecha del cuadro de texto de búsqueda en la parte superior de la página
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.

   >**Nota**: si es la primera vez que inicias **Cloud Shell** y aparece el mensaje **No tienes ningún almacenamiento montado**, selecciona la suscripción que utilizas en este laboratorio y haz clic en **Crear almacenamiento**.

1. En el símbolo del sistema de **Bash**, en el panel de **Cloud Shell**, ejecuta los siguientes comandos para recuperar los valores del atributo de identificador de suscripción de Azure:

    ```bash
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Nota**: copia ambos valores en un archivo de texto. Los necesitará más adelante en este laboratorio.

1. En el símbolo del sistema de **Bash**, en el panel **Cloud Shell**, ejecuta el siguiente comando para crear una entidad de servicio:

    ```bash
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Nota**: el comando generará una salida JSON. Copie los resultados en un archivo de texto. Lo necesitará más adelante en este laboratorio.

1. Después, desde el equipo del laboratorio, abre un explorador web y ve al proyecto **eShopOnWeb** de Azure DevOps. Haz clic en **Configuración del proyecto>Conexiones de servicio (en Canalizaciones)** y en **Nueva conexión de servicio**.
1. En la hoja **Nueva conexión de servicio**, selecciona **Administrador de recursos de Azure** y luego **Siguiente** (quizá debas desplazarte hacia abajo).
1. Elige **Entidad de servicio (manual)** y haz clic en **Siguiente**.
1. Rellena los campos vacíos con la información recopilada durante los pasos anteriores:
    - Identificador y nombre de la suscripción.
    - Identificador de entidad de servicio (appId), clave de entidad de servicio (contraseña) e identificador de inquilino (inquilino).
    - En **Nombre de conexión de servicio**, escribe **azure-connection**. Se hará referencia a este nombre en las canalizaciones de YAML cuando necesites una conexión de servicio de Azure DevOps para comunicarte con la suscripción de Azure.

1. Haz clic en **Comprobar y guardar**.

### Ejercicio 2: Importación y ejecución de la canalización de CI

En este ejercicio, importarás y ejecutarás la canalización de CI.

#### Tarea 1: importar y ejecutar la canalización de CI

1. Ve a **Canalizaciones>Canalizaciones**
1. Haga clic en el botón **Nueva canalización** (o **Crear canalización** si no tiene otras canalizaciones creadas previamente)
1. Selecciona **Git de Azure Repos (YAML)**
1. Selecciona el repositorio **eShopOnWeb**
1. Seleccione **Archivo YAML de Azure Pipelines existente**
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-ci-docker.yml**, y haga clic en **Continuar**
1. En la definición de canalización de YAML, personaliza los siguientes elementos:
   - **YOUR-SUBSCRIPTION-ID** por el identificador de la suscripción de Azure.
   - **rg-az400-container-NAME** con el nombre del grupo de recursos que creará la canalización (también puede ser un grupo de recursos existente).

1. Haz clic en **Guardar y ejecutar** y espera a que la canalización se ejecute correctamente.

    > **Nota**: la implementación puede tardar unos minutos en completarse.

    La definición de CI comprende las tareas siguientes:
    - **Recursos**: descarga los archivos del repositorio que se usarán en las siguientes tareas.
    - **AzureResourceManagerTemplateDeployment**: implementa Azure Container Registry mediante la plantilla de bicep.
    - **PowerShell**: recupera el valor del **servidor de inicio de sesión de ACR** de la salida de la tarea anterior y crea un nuevo parámetro **acrLoginServer**.
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines)**- Compilación**: compila la imagen de Docker y crea de dos etiquetas (la última y BuildID actual)
    - **Docker - Push**: inserta la imagen de Docker en Azure Container Registry.

1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar de nombre o mover**. Asígnale el nombre **eshoponweb-ci-docker** y haz clic en **Guardar**.

1. Ve a [**Azure Portal**](https://portal.azure.com) y busca Azure Container Registry en el grupo de recursos creado recientemente (debe denominarse **rg-az400-container-NAME**). En el lado izquierdo, haz clic en **Repositorios** en **Servicios** y asegúrate de que se ha creado el repositorio **eshoponweb/web**. Al hacer clic en el vínculo del repositorio, deberías ver dos etiquetas (una de ellas es **más reciente**), estas son las imágenes insertadas. Si no ves esto, comprueba el estado de la canalización.

### Ejercicio 3: Importación y ejecución de la canalización de CD

En este ejercicio, configurarás la conexión de servicio con la suscripción de Azure y después importarás y ejecutarás la canalización de CD.

#### Tarea 1: Agregar una nueva asignación de roles

En esta tarea, agregarás una nueva asignación de roles para permitir que App de Azure Service extraiga la imagen de Docker de Azure Container Registry.

1. Vaya a [**Azure Portal**](https://portal.azure.com).
1. En el portal de Azure portal, haz clic en el icono de **Cloud Shell**, situado inmediatamente a la derecha del cuadro de texto de búsqueda en la parte superior de la página
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.
1. En el símbolo del sistema de **Bash**, en el panel de **Cloud Shell**, ejecuta los siguientes comandos para recuperar los valores del atributo de identificador de suscripción de Azure:

    ```sh
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionId
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query "[0].name" --output tsv)
    echo $roleName
    ```

1. Después de obtener el identificador de la entidad de servicio y el nombre del rol, vamos a crear la asignación de roles ejecutando este comando (reemplaza **&lt;rg-az400-container-NAME&gt;** por el nombre del grupo de recursos).

    ```sh
    az role assignment create --assignee $spId --role $roleName --scope /subscriptions/$subscriptionId/resourceGroups/<rg-az400-container-NAME>
    ```

Ahora deberías ver la salida JSON que confirma el éxito de la ejecución del comando.

#### Tarea 2: Importar y ejecutar la canalización de CD

En esta tarea, importarás y ejecutarás la canalización de CD.

1. Ve a **Canalizaciones>Canalizaciones**
1. Haz clic en el botón **Nueva canalización**.
1. Selecciona **Git de Azure Repos (YAML)**
1. Selecciona el repositorio **eShopOnWeb**
1. Selecciona **Archivo YAML de Azure Pipelines existente**
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-cd-webapp-docker.yml**, y haga clic en **Continuar**
1. En la definición de canalización de YAML, personaliza los siguientes elementos:
   - **YOUR-SUBSCRIPTION-ID** por el identificador de la suscripción de Azure.
   - **rg-az400-container-NAME** con el nombre del grupo de recursos definido antes en el laboratorio.

1. Haz clic en **Guardar y ejecutar** y espera a que la canalización se ejecute correctamente.

    > **Nota**: la implementación puede tardar unos minutos en completarse.

    > **Importante**: si recibes el mensaje de error “TF402455: No se permiten inserciones a esta rama; debes usar una solicitud de incorporación de cambios para actualizar esta rama”. Debes desactivar la regla de protección de rama “Requerir un número mínimo de revisores” habilitada en los laboratorios anteriores.

    La definición de CD consta de las siguientes tareas:
    - **Recursos**: descarga los archivos del repositorio que se usarán en las siguientes tareas.
    - **AzureResourceManagerTemplateDeployment**: implementa Azure App Service mediante la plantilla de bicep.
    - **AzureResourceManagerTemplateDeployment**: agregar asignación de roles mediante Bicep

1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y mantén el puntero sobre la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar de nombre o mover**. Asígnale el nombre **eshoponweb-cd-webapp-docker** y haz clic en **Guardar**.

    > **Nota 1**: El uso de la plantilla **/infra/webapp-docker.bicep** crea un plan de App Service, una aplicación web con la identidad administrada asignada por el sistema habilitada y hace referencia a la imagen de Docker insertada anteriormente: **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Nota 2**: El uso de la plantilla **/infra/webapp-to-acr-roleassignment.bicep** crea una nueva asignación de roles para la aplicación web con el rol AcrPull para poder recuperar la imagen de Docker. Esto puede hacerse en la primera plantilla, pero dado que la asignación de roles puede tardar algún tiempo en propagarse, es una buena idea realizar ambas tareas por separado.

#### Tarea 3: Probar la solución

1. En Azure Portal, ve al grupo de recursos creado recientemente. Ahora deberías ver tres recursos (App Service, Plan de App Service y Container Registry).

1. Ve a App Service y luego haz clic en **Examinar**, lo que te llevará al sitio web.

Felicidades. En este ejercicio, has implementado un sitio web mediante una imagen personalizada de Docker.

### Ejercicio 4: Eliminación de los recursos del laboratorio de Azure

En este ejercicio, quitarás los recursos de Azure aprovisionados en este laboratorio para eliminar cargos inesperados.

>**Nota**: No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

#### Tarea 1: eliminar los recursos del laboratorio de Azure

En esta tarea, usarás Azure Cloud Shell para quitar los recursos de Azure aprovisionados en este laboratorio con el propósito de eliminar cargos innecesarios.

1. En Azure Portal, abra la sesión de shell de **Bash** en el panel **Cloud Shell**.
1. Ejecute el comando siguiente para enumerar todos los grupos de recursos que se han creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
    ```

1. Ejecute el comando siguiente para eliminar todos los grupos de recursos que ha creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro --nowait). Aunque podrá ejecutar otro comando de la CLI de Azure inmediatamente después en la misma sesión de Bash, los grupos de recursos tardarán unos minutos en quitarse.

## Revisar

En este laboratorio, has aprendido a usar una canalización de CI/CD de Azure DevOps para compilar una imagen de Docker personalizada, insertarla en Azure Container Registry e implementarla como un contenedor en Azure App Service.
