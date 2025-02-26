---
lab:
  title: Configuración de canalizaciones como código con YAML
  module: 'Module 03: Design and implement a release strategy'
---

# Configuración de canalizaciones como código con YAML

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique una suscripción de Azure existente o cree una.

- Compruebe que tiene una cuenta Microsoft o una cuenta de Microsoft Entra con el rol Propietario en la suscripción de Azure y el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Introducción al laboratorio

Muchos equipos prefieren definir sus canalizaciones de compilación y versión mediante YAML. Esto les permite acceder a las mismas características de canalización que las que usan el diseñador visual, pero con un archivo de marcado que se puede administrar como cualquier otro archivo de código fuente. Las definiciones de compilación de YAML se pueden agregar a un proyecto agregando simplemente los archivos correspondientes a la raíz del repositorio. Azure DevOps también proporciona plantillas predeterminadas para tipos de proyecto populares y un diseñador YAML para simplificar el proceso de definición de tareas de compilación y versión.

## Objetivos

Después de completar este laboratorio, podrá:

- Configure canalizaciones de CI/CD como código con YAML en Azure DevOps.

## Tiempo estimado: 45 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio.

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb_MultiStageYAML** Azure DevOps que usarán varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asigna al proyecto el nombre **eShopOnWeb_MultiStageYAML** y deja el resto de los campos con los valores predeterminados. Haga clic en **Crear**.

   ![Captura de pantalla del panel Crear nuevo proyecto.](images/create-project.png)

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo de laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto de **eShopOnWeb_MultiStageYAML creado **anteriormente. Haz clic en **Repos > Archivos**, **Importar un repositorio**. Seleccione **importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL https://github.com/MicrosoftLearning/eShopOnWeb.git y haz clic en **Importar**:

   ![Captura de pantalla del panel Importar repositorio.](images/import-repo.png)

1. El repositorio se organiza de la siguiente manera:
   - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
   - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
   - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
   - Definiciones de flujo de trabajo de GitHub del contenedor de carpetas **.github**.
   - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

1. Ve a **Repos > Ramas**.
1. Mantén el puntero sobre la rama **main** y haz clic en los puntos suspensivos a la derecha de la columna.
1. Haz clic en **Establecer como rama predeterminada**.

#### Tarea 3: Creación de recursos de Azure

En este ejercicio, se usa Azure Portal para crear una aplicación web de Azure.

1. En el equipo del laboratorio, abre un explorador web, ve al [**Portal de Azure**](https://portal.azure.com) e inicia sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que vas a usar en este laboratorio, así como el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción.
1. Haz clic en el icono de la barra de herramientas a la derecha del cuadro de texto de búsqueda en Azure Portal para abrir el panel de **Cloud Shell**.
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.

   > **Nota**: si es la primera vez que inicias **Cloud Shell** y aparece el mensaje **No tienes ningún almacenamiento montado**, selecciona la suscripción que utilizas en este laboratorio y haz clic en **Crear almacenamiento**.

   > **Nota:** Para obtener una lista de regiones y sus alias, ejecuta el siguiente comando desde Azure Cloud Shell - Bash:

   ```bash
   az account list-locations -o table
   ```

1. En el símbolo del **sistema de Bash**, en el panel de **Cloud Shell**, ejecuta el siguiente comando para crear un grupo de recursos (reemplaza el `<region>` marcador de posición por el nombre de la región de Azure más cercana, como “centralus”, “westeurope” u otra región que prefieras).

   ```bash
   LOCATION='<region>'
   ```

   ```bash
   RESOURCEGROUPNAME='az400m03l07-RG'
   az group create --name $RESOURCEGROUPNAME --location $LOCATION
   ```

1. Ejecuta el comando siguiente para crear un plan de Windows App Service:

   ```bash
   SERVICEPLANNAME='az400m03l07-sp1'
   az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
   ```

1. Crea una nueva aplicación web con un nombre único.

   ```bash
   WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
   az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
   ```

   > **Nota**: registra el nombre de la aplicación web. Lo necesitará más adelante en este laboratorio.

1. Cierra Azure Cloud Shell, pero deja Azure Portal abierto en el explorador.

### Ejercicio 1: configuración de canalizaciones de CI/CD como código con YAML en Azure DevOps

En este ejercicio, configurarás las canalizaciones de CI/CD como código con YAML en Azure DevOps.

#### Tarea 1: Agregar una definición de compilación de YAML

En esta tarea, agregarás una definición de compilación de YAML al proyecto existente.

1. Vuelve al panel **Canalizaciones** del centro de **Canalizaciones**.
1. En la ventana **Crear la primera canalización**, haz clic en **Crear canalización**.

   > **Nota**: usaremos el asistente para crear una nueva definición de canalización de YAML basada en nuestro proyecto.

1. En el panel **¿Dónde está el código?**, haz clic en la opción **Azure Repos Git (YAML)**.
1. En el panel **Seleccionar un repositorio**, haz clic en **eShopOnWeb_MultiStageYAML**.
1. En el panel **Configurar tu canalización**, desplázate hacia abajo y selecciona **Archivo YAML de Azure Pipelines existente**.
1. En la hoja **Seleccionar un archivo YAML existente**, especifica los parámetros siguientes:
   - Rama: **principal**.
   - Ruta de acceso: **.ado/eshoponweb-ci.yml**
1. Haz clic en **Continuar** para guardar esta configuración.
1. En la pantalla **Revisar YAML de canalización**, haz clic en **Ejecutar** para iniciar el proceso de canalización de compilación.
1. Espera a que la canalización de compilación se complete correctamente. Omite las advertencias relacionadas con el propio código fuente, ya que no son pertinentes para este ejercicio de laboratorio.

   > **Nota**: Cada tarea del archivo YAML está disponible para su revisión, incluidas las advertencias y errores.

#### Tarea 2: Agregar entrega continua a la definición de YAML

En esta tarea, agregarás la entrega continua a la definición basada en YAML de la canalización que creaste en la tarea anterior.

> **Nota**: Ahora que los procesos de compilación y prueba son correctos, podemos agregar la entrega a la definición de YAML.

1. En el panel de ejecución de la canalización, haz clic en los puntos suspensivos en la esquina superior derecha y, en el menú desplegable, haz clic en **Editar canalización**.
1. En el panel que muestra el contenido del archivo **eShopOnWeb_MultiStageYAML/.ado/eshoponweb-ci.yml**, navega hasta el final del archivo (línea 56) y presiona **Entrar/Retorno** para agregar una nueva línea vacía.
1. En la línea **57**, agrega el siguiente contenido para definir la fase de la **Versión** en la canalización de YAML.

   > **Nota**: Puedes definir las fases que necesites para organizar y hacer un seguimiento del progreso de la canalización.

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - job: Deploy
         pool:
           vmImage: "windows-latest"
         steps:
   ```

1. Establece el cursor en una nueva línea al final de la definición de YAML.

   > **Nota**: aquí se agregarán nuevas tareas.

1. En la lista de tareas a la derecha del panel de código, busca y selecciona la tarea **Implementación de Azure App Service**.
1. En el panel **Implementación de Azure App Service**, especifica la siguiente configuración y haz clic en **Agregar**:

   - En la lista desplegable **Suscripción a Azure**, selecciona la suscripción en la que has implementado los recursos de Azure en el laboratorio, haz clic en **Autorizar** y, cuando se solicite, ingresa con las credenciales de la misma cuenta que has usado durante la implementación de recursos de Azure.
   - En la lista desplegable **Nombre de App Service**, selecciona el nombre de la aplicación web que has implementado en el laboratorio.
   - En el cuadro de texto **Paquete o carpeta**, **actualiza** el valor predeterminado a `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
   - En **Configuración y opciones de la aplicación**, agregue `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development`.

1. Para confirmar la configuración del panel Asistente, haz clic en el botón **Agregar**.

   > **Nota**: esto agregará automáticamente la tarea de implementación a la definición de canalización de YAML.

1. El fragmento de código agregado al editor debe ser similar al siguiente, lo que refleja el nombre de los parámetros azureSubscription y WebappName:

   ```yaml
   - task: AzureRmWebAppDeployment@4
     inputs:
       ConnectionType: "AzureRM"
       azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
       appType: "webApp"
       WebAppName: "eshoponWebYAML369825031"
       packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
       AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. Verifica si la tarea aparece como elemento secundario de la tarea **Pasos**. Si no es así, selecciona todas las líneas de la tarea agregada, presiona la tecla **Tabulador** dos veces para que haya una sangría de cuatro espacios, de forma que aparezca como elemento secundario de la tarea **Pasos**.

   > **Nota**: el parámetro **packageForLinux** es confuso en este laboratorio, pero es válido para Windows o Linux.

   > **Nota**: De forma predeterminada, estas dos fases se ejecutan de forma independiente. Como resultado, es posible que la salida de compilación de la primera fase no esté disponible en la segunda fase sin cambios adicionales. Para implementar estos cambios, agregaremos una nueva tarea para descargar el artefacto de implementación al principio de la fase de implementación.

1. Coloca el cursor en la primera línea bajo el nodo **steps** de la fase **implementación** y presiona Entrar/Retorno para agregar una nueva línea vacía (línea 64).
1. En el panel **Tareas**, busca y selecciona la tarea **Descargar artefactos de compilación**.
1. Especifica los siguientes parámetros para esta tarea:
   - Descarga de los artefactos producidos por: **Compilación actual**
   - Tipo de descarga: **Artefacto específico**
   - Nombre de artefacto: **selecciona "Sitio web" en la lista** (o **escribe "`Website`"** directamente si no aparece automáticamente en la lista)
   - Directorio de destino: **$(Build.ArtifactStagingDirectory)**
1. Haga clic en **Agregar**.
1. El fragmento de código agregado debe tener un aspecto similar al siguiente:

   ```yaml
   - task: DownloadBuildArtifacts@1
     inputs:
       buildType: "current"
       downloadType: "single"
       artifactName: "Website"
       downloadPath: "$(Build.ArtifactStagingDirectory)"
   ```

1. Si la sangría de YAML está desactivada, con la tarea agregada aún seleccionada en el editor, presiona la tecla **Tabulador** dos veces para sangría en cuatro espacios.

   > **Nota**: aquí puede que también quieras agregar una línea vacía antes y después para facilitar la lectura.

1. Haz clic en **Guardar**. En el panel **Guardar**, haz clic en **Guardar** de nuevo para confirmar el cambio directamente en la rama principal.

   > **Nota**: dado que el sistema CI-YAML original no se ha configurado para desencadenar automáticamente una nueva compilación, tenemos que iniciarla manualmente.

1. En el menú a la izquierda de Azure DevOps, ve a la pestaña **Canalizaciones** y selecciona **Canalizaciones**.
1. Abra la canalización de **eShopOnWeb_MultiStageYAML** y haga clic en **Ejecutar canalización**.
1. Confirma la opción **Ejecutar** desde el panel que aparece.
1. Verás que aparecen dos fases diferentes: **Compilar una solución .Net Core** e **Implementar en una aplicación web de Azure**.
1. Espera a que se inicie la canalización y a que finalice la fase de compilación.
1. Cuando la fase de implementación quiera iniciarse, deberás introducir **los permisos necesarios** y verás una barra naranja que dice:

   ```text
   This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
   ```

1. Haz clic en **Ver**.
1. En el panel **Esperando revisión**, haz clic en **Permitir**.
1. Valida el mensaje en la ventana **Permitir ventana emergente** y confirma haciendo clic en **Permitir**.
1. Esto activa la fase de implementación. Espera a que la implementación se complete correctamente.

   > **Nota**: si se debe producir un error en la implementación, debido a un problema con la sintaxis de canalización de YAML, usa esto como referencia:

   ```yaml
   #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
   # trigger:
   # - main

   resources:
    repositories:
      - repository: self
        trigger: none

  stages:
  - stage: Build
    displayName: Build .Net Core Solution
    jobs:
    - job: Build
      pool:
        vmImage: ubuntu-latest
      steps:
      - task: DotNetCoreCLI@2
        displayName: Restore
        inputs:
          command: 'restore'
          projects: '**/*.sln'
          feedsToUse: 'select'

      - task: DotNetCoreCLI@2
        displayName: Build
        inputs:
          command: 'build'
          projects: '**/*.sln'

      - task: DotNetCoreCLI@2
        displayName: Test
        inputs:
          command: 'test'
          projects: 'tests/UnitTests/*.csproj'

      - task: DotNetCoreCLI@2
        displayName: Publish
        inputs:
          command: 'publish'
          publishWebProjects: true
          arguments: '-o $(Build.ArtifactStagingDirectory)'

      - task: PublishBuildArtifacts@1
        displayName: Publish Artifacts ADO - Website
        inputs:
          pathToPublish: '$(Build.ArtifactStagingDirectory)'
          artifactName: Website

      - task: PublishBuildArtifacts@1
        displayName: Publish Artifacts ADO - Bicep
        inputs:
          PathtoPublish: '$(Build.SourcesDirectory)/infra/webapp.bicep'
          ArtifactName: 'Bicep'
          publishLocation: 'Container'

   - stage: Deploy
    displayName: Deploy to an Azure Web App
    jobs:
    - job: Deploy
      pool:
        vmImage: 'windows-latest'
      steps:
      - task: DownloadBuildArtifacts@0
        inputs:
          buildType: 'current'
          downloadType: 'single'
          artifactName: 'Website'
          downloadPath: '$(Build.ArtifactStagingDirectory)'
      - task: AzureRmWebAppDeployment@4
        inputs:
          ConnectionType: 'AzureRM'
          azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
          appType: 'webApp'
          WebAppName: 'eshoponWebYAML369825031'
          packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
          AppSettings: '-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development'

   ```

#### Tarea 3: Revisión del sitio implementado

1. Vuelve a la ventana del explorador web que muestra el portal de Azure y ve a la hoja que muestra las propiedades de la aplicación web de Azure.
1. En la hoja de aplicación web de Azure, haz clic en **Información general** y, en la hoja de información general, haz clic en **Examinar** para abrir el sitio en una nueva pestaña del explorador web.
1. Compruebe que el sitio implementado se carga según lo previsto en la nueva pestaña del explorador, que muestra el sitio web de comercio electrónico eShopOnWeb.

### Ejercicio 2: Configurar canalizaciones de CI/CD como código con YAML en Azure DevOps

En este ejercicio, agregarás aprobaciones a una canalización basada en YAML en Azure DevOps.

#### Tarea 1: Configurar entornos de canalización

Las canalizaciones de YAML como código no tienen puertas de versión o calidad, como tenemos con canalizaciones de versión clásicas de Azure DevOps. Pero algunas similitudes se pueden configurar para canalizaciones YAML como código mediante **entornos**. En esta tarea, usarás este mecanismo para configurar aprobaciones para la fase de compilación.

1. En el proyecto de Azure DevOps **eShopOnWeb_MultiStageYAML**, vaya a **Canalizaciones**.
1. En el menú Canalizaciones de la izquierda, selecciona **Entornos**.
1. Haz clic en **Crear entorno**.
1. En el panel **Nuevo entorno**, agrega un nombre para el entorno, denominado **`approvals`**.
1. En **Recursos**, selecciona **Ninguno**.
1. Confirma la configuración presionando el botón **Crear**.
1. Una vez creado el entorno, selecciona la pestaña **Aprobaciones y comprobaciones** en el entorno **Aprobaciones**.
1. En la casilla **Agregue su primera comprobación**, selecciona **Aprobaciones**.
1. Agrega el nombre de la cuenta de usuario de Azure DevOps al campo **aprobadores**.

   > **Nota:** en un escenario real, esto reflejaría el nombre del equipo de DevOps que trabaja en este proyecto.

1. Confirma la configuración de aprobación definida presionando el botón **Crear**.
1. Por último, es necesario agregar la configuración necesaria de "entorno: aprobaciones" al código de canalización de YAML para la fase de implementación. Para ello, ve a **Repos**, ve a la carpeta **.ado** y selecciona el archivo **eshoponweb-ci.yml**.
1. En la vista Contenido, haz clic en el botón **Editar** para cambiar al modo de edición.
1. Ve al inicio del **Trabajo de implementación** (-job: Deploy on Line 60)
1. Agrega una nueva línea vacía justo debajo y agrega el siguiente fragmento de código:

   ```yaml
   environment: approvals
   ```

   El fragmento de código resultante debe tener este aspecto:

   ```yaml
   jobs:
     - job: Deploy
       environment: approvals
       pool:
         vmImage: "windows-latest"
   ```

1. Dado que el entorno es una configuración específica de una fase de implementación, no se puede usar en "trabajos". Por lo tanto, tenemos que realizar algunos cambios adicionales en la definición de trabajo actual.
1. En la línea **60**, cambia el nombre "- job: Deploy" por **- deployment: Deploy**
1. Después, en la línea **63** (vmImage: windows-latest), agrega una nueva línea vacía.
1. Pega el siguiente código YAML:

   ```yaml
   strategy:
     runOnce:
       deploy:
   ```

1. Selecciona el fragmento de código restante (línea **67** hasta el final) y usa la tecla **Tab** para corregir la marca de YAML.

   El fragmento de código YAML resultante ahora debería tener este aspecto, lo que refleja la **fase de implementación**:

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - deployment: Deploy
         environment: approvals
         pool:
           vmImage: "windows-latest"
         strategy:
           runOnce:
             deploy:
               steps:
                 - task: DownloadBuildArtifacts@1
                   inputs:
                     buildType: "current"
                     downloadType: "single"
                     artifactName: "Website"
                     downloadPath: "$(Build.ArtifactStagingDirectory)"
                 - task: AzureRmWebAppDeployment@4
                   inputs:
                     ConnectionType: "AzureRM"
                     azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
                     appType: "webApp"
                     WebAppName: "eshoponWebYAML369825031"
                     packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
                     AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. Confirma los cambios en el archivo YAML de código haciendo clic en **Confirmar** y haciendo clic en **Confirmar de nuevo** en el panel Confirmar que aparece.
1. Ve al menú Proyecto de Azure DevOps a la izquierda, selecciona **Canalizaciones**, canalizaciones**** y observa el **EshopOnWeb_MultiStageYAML** Canalización usada anteriormente.
1. Abre la canalización.
1. Haz clic en **Ejecutar canalización** para desencadenar una nueva ejecución de canalización; para ello, haz clic en **Ejecutar**.
1. Al igual que antes, la fase de compilación comienza según lo previsto. Espera a que la implementación se complete correctamente.
1. A continuación, dado que tenemos el _entorno: aprobaciones_ configuradas para la fase de implementación, solicitarás una confirmación de aprobación antes de que se inicie.
1. Esto es visible desde la vista Canalización, donde indica **Esperando (0/1 comprobaciones pasadas).** También se muestra un mensaje de notificación que indica que **la aprobación necesita revisión antes de que esta ejecución pueda continuar con la Implementación en una aplicación web de Azure**.
1. Haz clic en el botón **Vista** situado junto a este mensaje.
1. En el panel que aparece **Comprobaciones y validaciones manuales para Implementar en Azure Web App**, haz clic en el **mensaje Aprobación en espera**.
1. Haga clic en **Approve** (Aprobar).
1. Esto permite que la fase de implementación inicie e implemente correctamente el código fuente de Azure Web App.

   > **Nota:** Aunque en este ejemplo solo se usan las aprobaciones, debes saber que las otras comprobaciones como Azure Monitor, la API REST, etc. se pueden usar de forma similar.

   > [!IMPORTANT]
   > Recuerda eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisión

En esta pestaña, has configurado las canalizaciones de CI/CD como código con YAML en Azure DevOps.
