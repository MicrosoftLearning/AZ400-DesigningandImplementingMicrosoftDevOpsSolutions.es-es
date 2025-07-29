---
lab:
  title: "Implementación de contenedores de Docker en aplicaciones web de Azure\_App Service"
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Implementación de contenedores de Docker en aplicaciones web de Azure App Service

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

## Tiempo estimado: 20 minutos

## Instrucciones

### Ejercicio 0: (omitir si se ha realizado) Configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y elige **Scrum** en la lista desplegable **Proceso del elemento de trabajo**. Haga clic en **Crear**.

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos > Archivos**, **Importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

1. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - Definiciones de flujo de trabajo de GitHub del contenedor de carpetas **.github**.
    - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

#### Tarea 3: (omitir si ya la has completado) Establecer la rama principal como rama predeterminada

1. Ve a **Repos > Ramas**.
1. Mantén el puntero sobre la rama **main** y haz clic en los puntos suspensivos a la derecha de la columna.
1. Haz clic en **Establecer como rama predeterminada**.

### Ejercicio 1: Importación y ejecución de la canalización de CI

En este ejercicio, configurarás la conexión de servicio con la suscripción de Azure y luego importarás y ejecutarás la canalización de CI.

#### Tarea 1: importación y ejecución de la canalización de CI

1. Ve a **Canalizaciones > Canalizaciones**
1. Haga clic en el botón **Nueva canalización** (o **Crear canalización** si no tiene otras canalizaciones creadas previamente)
1. Selecciona **Git de Azure Repos (YAML)**
1. Selecciona el repositorio **eShopOnWeb**
1. Seleccione **Archivo YAML de Azure Pipelines existente**
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-ci-docker.yml**, y haga clic en **Continuar**
1. En la definición de canalización de YAML, personaliza los siguientes elementos:
   - **YOUR-SUBSCRIPTION-ID** por el identificador de la suscripción de Azure.
   - Reemplaza **resourceGroup** por el nombre del grupo de recursos usado durante la creación de la conexión de servicio, por ejemplo, **AZ400-RG1**.

1. Revisa la definición de la canalización. La definición de CI comprende las tareas siguientes:
    - **Recursos**: descarga los archivos del repositorio que se usarán en las siguientes tareas.
    - **AzureResourceManagerTemplateDeployment**: implementa Azure Container Registry mediante la plantilla de bicep.
    - **PowerShell**: recupera el valor del **servidor de inicio de sesión de ACR** de la salida de la tarea anterior y crea un nuevo parámetro **acrLoginServer**.
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines)**- Compilación**: compila la imagen de Docker y crea de dos etiquetas (la última y BuildID actual)
    - **Docker - Push**: inserta la imagen de Docker en Azure Container Registry.

1. Haz clic en **Guardar y ejecutar**.

1. Abre la ejecución de la canalización. Si ves el mensaje de advertencia "Esta canalización necesita permisos para acceder a un recurso antes de que la ejecución pueda continuar con la compilación", haz clic en **Ver**, **Permitir** y **Permitir** de nuevo. Esto permitirá que la canalización acceda a la suscripción a Azure.

    > **Nota**: la implementación puede tardar unos minutos en completarse.

1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones > Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar de nombre o mover**. Asígnale el nombre **eshoponweb-ci-docker** y haz clic en **Guardar**.

1. Ve a [**Azure Portal**](https://portal.azure.com) y busca Azure Container Registry en el grupo de recursos creado recientemente (debe denominarse **AZ400-RG1**). En el lado izquierdo, haz clic en **Repositorios** en **Servicios** y asegúrate de que se ha creado el repositorio **eshoponweb/web**. Al hacer clic en el vínculo del repositorio, deberías ver dos etiquetas (una de ellas es **más reciente**), estas son las imágenes insertadas. Si no ves esto, comprueba el estado de la canalización.

### Ejercicio 2: Importación y ejecución de la canalización de CD

En este ejercicio, configurarás la conexión de servicio con la suscripción de Azure y después importarás y ejecutarás la canalización de CD.

#### Tarea 1: Importación y ejecución de la canalización de CD

En esta tarea, importarás y ejecutarás la canalización de CD.

1. Ve a **Canalizaciones > Canalizaciones**
1. Haz clic en el botón **Nueva canalización**.
1. Selecciona **Git de Azure Repos (YAML)**
1. Selecciona el repositorio **eShopOnWeb**
1. Selecciona **Archivo YAML de Azure Pipelines existente**
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-cd-webapp-docker.yml**, y haga clic en **Continuar**
1. En la definición de canalización de YAML, personaliza los siguientes elementos:
   - **YOUR-SUBSCRIPTION-ID** por el identificador de la suscripción de Azure.
   - Reemplaza **resourceGroup** por el nombre del grupo de recursos usado durante la creación de la conexión de servicio, por ejemplo, **AZ400-RG1**.
   - Reemplaza la **ubicación** por la región de Azure en la que se implementarán los recursos.

1. Revisa la definición de la canalización. La definición de CD consta de las siguientes tareas:
    - **Recursos**: descarga los archivos del repositorio que se usarán en las siguientes tareas.
    - **AzureResourceManagerTemplateDeployment**: implementa Azure App Service mediante la plantilla de bicep.
    - **AzureResourceManagerTemplateDeployment**: agregar asignación de roles mediante Bicep

1. Haz clic en **Guardar y ejecutar**.

1. Abre la ejecución de la canalización. Si ves el mensaje de advertencia "Esta canalización necesita permisos para acceder a un recurso antes de que la ejecución pueda continuar con la implementación", haz clic en **Ver**, **Permitir** y **Permitir** de nuevo. Esto permitirá que la canalización acceda a la suscripción a Azure.

    > **Importante**: Si no autorizas la canalización al realizar la configuración, se producirán errores de permiso durante la ejecución. Los mensajes de error comunes incluyen "Esta canalización necesita permiso para acceder a un recurso" o "Error de ejecución de canalización debido a permisos insuficientes". Para resolver estos errores, ve a la ejecución de la canalización, haz clic en **Ver** junto a la solicitud de permisos y, a continuación, haz clic en **Permitir** para conceder el acceso necesario a la suscripción y los recursos de Azure.

    > **Nota**: la implementación puede tardar unos minutos en completarse.

    > [!IMPORTANT]
    > Si recibes el mensaje de error "TF402455: No se permiten las inserciones en esta rama; debes usar una solicitud de incorporación de cambios para actualizar esta rama", debes desactivar la regla de protección de rama "Requerir un número mínimo de revisores" habilitada en los laboratorios anteriores.

1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones > Canalizaciones** y mantén el puntero sobre la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar de nombre o mover**. Asígnale el nombre **eshoponweb-cd-webapp-docker** y haz clic en **Guardar**.

    > **Nota 1**: El uso de la plantilla **/infra/webapp-docker.bicep** crea un plan de App Service, una aplicación web con la identidad administrada asignada por el sistema habilitada y hace referencia a la imagen de Docker insertada anteriormente: **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Nota 2**: El uso de la plantilla **/infra/webapp-to-acr-roleassignment.bicep** crea una nueva asignación de roles para la aplicación web con el rol AcrPull para poder recuperar la imagen de Docker. Esto puede hacerse en la primera plantilla, pero dado que la asignación de roles puede tardar algún tiempo en propagarse, es una buena idea realizar ambas tareas por separado.

#### Tarea 2: Prueba de la solución

1. En Azure Portal, ve al grupo de recursos creado recientemente. Ahora deberías ver tres recursos (App Service, Plan de App Service y Container Registry).

1. Ve a App Service y luego haz clic en **Examinar**, lo que te llevará al sitio web.

1. Comprueba que la aplicación eShopOnWeb se está ejecutando correctamente. Una vez confirmado, has completado correctamente el laboratorio.

> [!IMPORTANT]
> Recuerda eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisión

En este laboratorio, has aprendido a usar una canalización de CI/CD de Azure DevOps para compilar una imagen de Docker personalizada, insertarla en Azure Container Registry e implementarla como un contenedor en Azure App Service.
