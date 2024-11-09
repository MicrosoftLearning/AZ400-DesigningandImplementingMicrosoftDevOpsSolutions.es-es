---
lab:
  title: Supervisión del rendimiento de las aplicaciones con Azure Load Testing
  module: 'Module 08: Implement continuous feedback'
---

# Supervisión del rendimiento de las aplicaciones con Azure Load Testing

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique una suscripción de Azure existente o cree una.

- Compruebe que tiene una cuenta Microsoft o una cuenta de Microsoft Entra con el rol Propietario en la suscripción de Azure y el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Introducción al laboratorio

**Azure Load Testing** es un servicio de prueba de carga totalmente administrado que permite generar una carga a gran escala. El servicio simula el tráfico de las aplicaciones, independientemente del lugar en que se hospeden. Los desarrolladores, evaluadores e ingenieros de control de calidad (QA) pueden usarlo para optimizar el rendimiento, la escalabilidad o la capacidad de las aplicaciones.
Cree rápidamente una prueba de carga para la aplicación web mediante una dirección URL y sin conocimientos previos de las herramientas de prueba. Azure Load Testing abstrae la complejidad y la infraestructura para ejecutar una prueba de carga a gran escala.
Para escenarios de prueba de carga más avanzados, puede crear una prueba de carga mediante la reutilización de un script de prueba existente de Apache JMeter, una popular herramienta de carga y rendimiento de código abierto. Por ejemplo, el plan de pruebas puede constar de varias solicitudes de aplicación, de llamadas a puntos de conexión no HTTP o del uso de parámetros y datos de entrada para que la prueba sea más dinámica.

En este laboratorio, obtendrás información sobre cómo usar Azure Load Testing para simular las pruebas de rendimiento en una aplicación web que se ejecuta en vivo con diferentes escenarios de carga. Por último, aprenderás a integrar Azure Load Testing en las canalizaciones de CI/CD.

## Objetivos

Después de completar este laboratorio, podrá:

- Implemente aplicaciones web de Azure App Service.
- Redacta y ejecuta una canalización de CI/CD basada en YAML.
- Implementa Azure Load Testing.
- Investiga el rendimiento de las aplicaciones web de Azure mediante Azure Load Testing.
- Integra Azure Load Testing y Azure Chaos Studio en las canalizaciones de CI/CD.

## Tiempo estimado: 60 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio.

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y elige **Scrum** en la lista desplegable **Proceso del elemento de trabajo**. Haga clic en **Crear**.

    ![Captura de pantalla del panel Crear nuevo proyecto.](images/create-project.png)

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos > Archivos**, **Importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

    ![Captura de pantalla del Importar panel de repositorio.](images/import-repo.png)

1. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** tiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - Definiciones de flujo de trabajo de GitHub del contenedor de carpetas **.github**.
    - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

#### Tarea 3: (omitir si ya la has completado) Establecer la rama principal como rama predeterminada

1. Ve a **Repos > Ramas**.
1. Mantén el puntero sobre la rama **main** y haz clic en los puntos suspensivos a la derecha de la columna.
1. Haz clic en **Establecer como rama predeterminada**.

#### Tarea 4: Creación de recursos de Azure

En esta tarea, crearás una aplicación web de Azure mediante Cloud Shell en Azure Portal.

1. En el equipo de laboratorio, inicia un explorador web, ve a [**Azure Portal**](https://portal.azure.com) e inicia sesión.
1. Haz clic en el icono de la barra de herramientas a la derecha del cuadro de texto de búsqueda en Azure Portal para abrir el panel de **Cloud Shell**.
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.
    > **Nota**: si es la primera vez que inicias **Cloud Shell** y aparece el mensaje **No tienes ningún almacenamiento montado**, selecciona la suscripción que utilizas en este laboratorio y haz clic en **Crear almacenamiento**.

1. En el símbolo del sistema de **Bash**, en el panel de **Cloud Shell**, ejecuta el siguiente comando para crear un grupo de recursos (reemplaza el marcador de posición `<region>` con el nombre de la región de Azure más cercana, como "eastus").

    ```bash
    RESOURCEGROUPNAME='az400m08l14-RG'
    LOCATION='<region>'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

1. Ejecuta el comando siguiente para crear un plan de Windows App Service:

    ```bash
    SERVICEPLANNAME='az400l14-sp'
    az appservice plan create --resource-group $RESOURCEGROUPNAME \
        --name $SERVICEPLANNAME --sku B3
    ```

1. Crea una nueva aplicación web con un nombre único.

    ```bash
    WEBAPPNAME=az400eshoponweb$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
    ```

    > **Nota**: registra el nombre de la aplicación web. Lo necesitará más adelante en este laboratorio.

### Ejercicio 1: configuración de canalizaciones de CI/CD como código con YAML en Azure DevOps

En este ejercicio, configurarás las canalizaciones de CI/CD como código con YAML en Azure DevOps.

#### Tarea 1: aregar una definición de compilación e implementación de YAML

En esta tarea, agregarás una definición de compilación de YAML al proyecto existente.

1. Vuelve al panel **Canalizaciones** del centro de **Canalizaciones**.
1. Haga clic en **Nueva canalización** (o en Crear canalización si es la primera que crea).

    > **Nota**: usaremos el asistente para crear una nueva definición de canalización de YAML basada en nuestro proyecto.

1. En el panel **¿Dónde está el código?**, haz clic en la opción **Azure Repos Git (YAML)**.
1. En el panel **Seleccionar un repositorio**, haz clic en **eShopOnWeb**.
1. En el panel **Configurar tu canalización**, desplázate hacia abajo y selecciona **Canalización inicial**.
1. **Selecciona** todas las líneas de la canalización de inicio y elimínalas.
1. **Copia** la canalización de plantilla completa de la siguiente manera, sabiendo que deberás modificar los parámetros **antes de guardar** los cambios:

    ```yml
    
    #Template Pipeline for CI/CD
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

    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
        - task: DownloadBuildArtifacts@1
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'

    ```

1. Establece el cursor en una nueva línea al final de la definición de YAML. **Asegúrese de colocar el cursor en la sangría del nivel**de la tarea anterior.

    > **Nota**: aquí se agregarán nuevas tareas.

1. Haga clic en **Mostrar asistente** en el lado derecho del portal. En la lista de tareas, busque la tarea **Implementación de Azure App Service** y selecciónela.
1. En el panel **Implementación de Azure App Service**, especifica la siguiente configuración y haz clic en **Agregar**:

    - en la lista desplegable **suscripción de Azure**, seleccione la conexión de servicio que acaba de crear.
    - Valide que **Tipo de App Service** apunta a Web App en Windows.
    - En la lista desplegable **Nombre de App Service**, seleccione el nombre de la aplicación web que ha implementado en este mismo laboratorio (**az400eshoponweb...).
    - En el cuadro de texto **Paquete o carpeta**, **actualiza** el valor predeterminado a `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
    - Expande **Configuración y ajustes de la aplicación** y, en el cuadro de texto Configuración de la aplicación, agrega los siguientes pares clave-valor: `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development`.
1. Para confirmar la configuración del panel Asistente, haz clic en el botón **Agregar**.

    > **Nota**: esto agregará automáticamente la tarea de implementación a la definición de canalización de YAML.

1. El fragmento de código agregado al editor debe ser similar al siguiente, lo que refleja el nombre de los parámetros azureSubscription y WebappName:

    ```yml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'SERVICE CONNECTION NAME'
            appType: 'webApp'
            WebAppName: 'az400eshoponWeb369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
            AppSettings: '-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development'
    ```

    > **Nota**: el parámetro **packageForLinux** es confuso en este laboratorio, pero es válido para Windows o Linux.

1. Antes de guardar las actualizaciones en el archivo yml, asígnele un nombre más claro. En la parte superior de la ventana del editor de yaml, aparece el nombre **EShopOnweb/azure-pipelines-#.yml**. (donde # es un número, normalmente 1, pero podría ser diferente en tu configuración). Selecciona **ese nombre de archivo y cámbialo **por **m08l14-pipeline.yml**

1. Haz clic en **Guardar**. En el panel **Guardar**, haz clic en **Guardar** de nuevo para confirmar el cambio directamente en la rama principal.

    > **Nota**: dado que el sistema CI-YAML original no se ha configurado para desencadenar automáticamente una nueva compilación, tenemos que iniciarla manualmente.

1. En el menú a la izquierda de Azure DevOps, ve a la pestaña **Canalizaciones** y selecciona **Canalizaciones**. Luego, seleccione **Todo** para abrir todas las definiciones de canalización, no solo las recientes.

    > **Nota**: si ha guardado todas las canalizaciones de los ejercicios de laboratorio anteriores, es posible que esta nueva canalización haya reutilizado el nombre de secuencia predeterminado **eShopOnWeb (#)** para la canalización, como se muestra en la captura de pantalla siguiente. Selecciona una canalización (si es posible, la que tenga el número de secuencia más alto, selecciona Editar y valida que apunta al archivo de código m08l14-pipeline.yml).

    ![Captura de pantalla de Azure Pipelines que muestra ejecuciones de eShopOnWeb.](images/m3/eshoponweb-m9l16-pipeline.png)

1. Para confirmar que esta canalización se ejecuta, haga clic en **Ejecutar** en el panel que aparece y haga clic una vez más en **Ejecutar** para confirmar la operación.
1. Verás que aparecen dos fases diferentes: **Compilar una solución .Net Core** e **Implementar en una aplicación web de Azure**.
1. Espere a que se inicie la canalización.

1. **Ignore** las advertencias que se muestran en la fase de compilación. Espere hasta que complete correctamente la fase de compilación. (puede seleccionar la fase de compilación real para ver más detalles en los registros).

1. Cuando la fase de implementación quiera iniciarse, deberás introducir **los permisos necesarios** y verás una barra naranja que dice:

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

1. Haz clic en **Ver**.
1. En el panel **Esperando revisión**, haz clic en **Permitir**.
1. Valida el mensaje en la ventana **Permitir ventana emergente** y confirma haciendo clic en **Permitir**.
1. Esto activa la fase de implementación. Espera a que la implementación se complete correctamente.

#### Tarea 2: revisar el sitio implementado

1. Vuelve a la ventana del explorador web que muestra el portal de Azure y ve a la hoja que muestra las propiedades de la aplicación web de Azure.
1. En la hoja de aplicación web de Azure, haz clic en **Información general** y, en la hoja de información general, haz clic en **Examinar** para abrir el sitio en una nueva pestaña del explorador web.
1. Compruebe que el sitio implementado se carga según lo previsto en la nueva pestaña del explorador, que muestra el sitio web de comercio electrónico eShopOnWeb.

### Ejercicio 2: implementación y configuración de Azure Load Testing

En este ejercicio, implementarás un recurso de Azure Load Testing en Azure y configurarás diferentes escenarios de pruebas de carga para Azure App Service en directo.

#### Tarea 1: implementar pruebas de carga de Azure

En esta tarea, implementarás un recurso de Azure Load Testing en la suscripción de Azure.

1. En Azure Portal (<https://portal.azure.com>), vaya a **Creación de recursos de Azure**.
1. En el campo de búsqueda "Buscar servicios y marketplace", escribe **`Azure Load Testing`**.
1. En los resultados, selecciona **Azure Load Testing** (publicado por Microsoft).
1. En la página Azure Load Testing, haz clic en **Crear** para iniciar el proceso de implementación.
1. En la página "Crear un recurso de prueba de carga", proporciona los detalles necesarios para la implementación de recursos:
   - **Suscripción**: selecciona la suscripción de Azure
   - **Grupo de recursos**: selecciona el grupo de recursos que ha usado para implementar Web App Service en el ejercicio anterior.
   - **Nombre**: `eShopOnWebLoadTesting`
   - **Región**: selecciona una región cercana a tu área

    > **Nota**: el servicio Azure Load Testing no está disponible en todas las regiones de Azure.

1. Para validar la configuración, haz clic en **Revisar y crear**.
1. Haz clic en **Crear** para confirmar y obtén el recurso Azure Load Testing implementado.
1. Pasas a la página "La implementación está en curso". La implementación tarda unos minutos en finalizar.
1. Haga clic en **Ir a al recurso** en la página de progreso de la implementación para ir al recurso **eShopOnWebLoadTesting** de Azure Load Testing.

    > **Nota**: si has cerrado la hoja o el portal de Azure durante la implementación del recurso  Azure Load Testing, puedes encontrarlo de nuevo desde el campo Búsqueda del portal de Azure o desde Recursos/Lista de recursos recientes.

#### Tarea 2: crear pruebas de Azure Load Testing

En esta tarea, crearás diferentes pruebas de Azure Load Testing con distintas opciones de configuración de carga.

1. En la hoja del recurso **eShopOnWebLoadTesting** de Azure Load Testing, ve a **Pruebas** en **Pruebas**. Haz clic en la opción de menú **+Crear** y selecciona **Crear una prueba basada en URL**.
1. Desactiva la casilla **Habilitar configuración avanzada** para mostrar la configuración avanzada.
1. Completa los parámetros y la configuración siguientes para crear una prueba de carga:
   - **URL de prueba**: escriba la dirección URL del servicio Azure App Service que implementó en el ejercicio anterior (az400eshoponweb... azurewebsites.net), **incluido https://**
   - **Especificar carga**: usuarios virtuales
   - **Número de usuarios virtuales**: 50
   - **Duración de la prueba (minutos)**: 5
   - **Tiempo de aumento (minutos):**  1

1. Para confirmar la configuración de la prueba, haga clic en **Revisar y crear**(no realice ningún cambio en las demás pestañas). Haga clic en **Crear** una vez más.
1. De esta forma se inician las pruebas de carga, que durarán 5 minutos.
1. Con la prueba en ejecución, vuelva a la página de recurso **eShopOnWebLoadTesting** de Azure Load Testing y vaya a **Pruebas**, seleccione **Pruebas** y vea una prueba **Get_eshoponweb...**
1. En el menú superior, haga clic en **Crear**, **Crear una prueba basada en URL** para crear una segunda prueba de carga.
1. Completa los siguientes parámetros y ajustes para crear otra prueba de carga:
   - **URL de prueba**: escriba la dirección URL del servicio Azure App Service que implementó en el ejercicio anterior (eShopOnWeb...azurewebsites.net), **incluyendo https://**
   - **Especificar carga**: solicitudes por segundo (RPS)
   - **Solicitudes por segundo (RPS)**: 100
   - **Tiempo de respuesta (milisegundos)**: 500
   - **Duración de la prueba (minutos)**: 5
   - **Tiempo de aumento (minutos):**  1

1. Para confirmar la configuración de la prueba, haga clic en **Revisar y crear**y en **Crear** una vez más.
1. La prueba durará unos 5 minutos.

#### Tarea 3: validar los resultados de Azure Load Testing

En esta tarea, validarás el resultado de una prueba de ejecución de Azure Load Testing.

Una vez completadas ambas pruebas rápidas, vamos a hacer algunos cambios en ellas y validar los resultados.

1. En **Azure Load Testing**, vaya a **Pruebas**. Seleccione cualquiera de las definiciones de la prueba para abrir una vista más detallada. Para ello, debe **hacer clic** en una de las pruebas. Esto redirige a la página de prueba más detallada. Desde aquí, puedes validar los detalles de las ejecuciones reales seleccionando **TestRun_mm/dd/yy-hh:hh** de la lista.
1. En la página **TestRun** detallada, identifique el resultado real de la simulación de Azure Load Testing. Algunos de los valores son:
   - Carga / Total de solicitudes
   - Duration
   - Tiempo de respuesta (Muestra el resultado en segundos, lo que refleja el tiempo de respuesta del percentil 90. Esto significa que, para el 90% de las solicitudes, el tiempo de respuesta estará dentro de los resultados especificados).
   - Rendimiento en solicitudes por segundo

1. A continuación, se representan muchos de estos valores en líneas gráficas del panel y tablas.
1. Dedica unos minutos a **comparar los resultados** de las pruebas simuladas entre sí, y a **identificar el impacto** de más usuarios en el rendimiento de App Service.

### Ejercicio 2: Automatización de una prueba de carga con CI/CD en Azure Pipelines

Empiece a automatizar las pruebas de carga en Azure Load Testing agregándolo a una canalización de CI/CD. Después de ejecutar una prueba de carga en Azure Portal, exporta los archivos de configuración y configura una canalización de CI/CD en Azure Pipelines (Acciones de GitHub tiene una función similar).

Después de completar este ejercicio, tendrás un flujo de trabajo de CI/CD configurado para ejecutar una prueba de carga con Azure Load Testing.

#### Tarea 1: Identificación de los detalles de conexión de servicio de Azure DevOps

En esta tarea, concederás los permisos necesarios a la conexión de servicio de Azure DevOps.

1. En **Azure DevOps Portal**(<https://aex.dev.azure.com>), vaya al proyecto **eShopOnWeb**.
1. Selecciona **Configuración del proyecto** en la esquina inferior izquierda.
1. En la sección **Canalizaciones**, selecciona **Conexiones de servicio**.
1. Observa la conexión de servicio, que tiene el nombre de la suscripción de Azure que has usado para implementar recursos de Azure al principio del ejercicio del laboratorio.
1. **Selecciona la conexión de servicio**. En la pestaña **Información general**, ve a **Detalles** y selecciona **Administrar roles de conexión de servicio**.
1. Esto redirige a Azure Portal, desde donde abre los detalles del grupo de recursos en la hoja control de acceso (IAM).

#### Tarea 2: Concesión de permisos al recurso de Azure Load Testing

Azure Load Testing usa RBAC de Azure para conceder permisos para realizar actividades específicas en el recurso de prueba de carga. Para ejecutar una prueba de carga desde la canalización de CI/CD, concede el rol **Colaborador de pruebas de carga** a la conexión de servicio de Azure DevOps.

1. Seleccione **+ Agregar** y **Agregar asignación de roles**.
1. En la pestaña **Rol**, seleccione **Colaborador de pruebas de carga** en la lista de roles de funciones de trabajo.
1. En la **pestaña Miembros**, elige **Seleccionar miembros**, busca y selecciona tu cuenta de usuario y haz clic en **Seleccionar**.
1. En la **pestaña Revisar + asignar**, seleccione **Revisar + asignar** para agregar la asignación de roles.

Ahora es posible usar la conexión de servicio en la definición de flujo de trabajo de Azure Pipelines para acceder al recurso de prueba de carga de Azure.

#### Tarea 3: Exportación de archivos de entrada de prueba de carga e Importación a Azure Repos

Para ejecutar una prueba de carga con Azure Load Testing en un flujo de trabajo de CI/CD, es necesario agregar la configuración de prueba de carga y los archivos de entrada del repositorio de control de código fuente. Si tiene una prueba de carga existente, puede descargar los valores de configuración y todos los archivos de entrada desde Azure Portal.

Realice los pasos siguientes para descargar los archivos de entrada de una prueba de carga existente en Azure Portal:

1. En el portal de **Azure**, ve al recurso de **Azure Load Testing**.
1. En el panel izquierdo, selecciona **pruebas** para ver la lista y selecciona **tu prueba**.
1. Seleccione los puntos suspensivos (**...**) que hay junto a la serie de pruebas con la que trabaje y, después, seleccione **Descargar archivo de entrada**.
1. El explorador descarga una carpeta comprimida que contiene los archivos de entrada de pruebas de carga.
1. Use cualquier herramienta zip para extraer los archivos de entrada. La carpeta contiene los archivos siguientes:

   - *config.yaml*: el archivo de configuración de YAML de prueba de carga. Haga referencia a este archivo en la definición de flujo de trabajo de CI/CD.
   - *quick_test.jmx*: el script de prueba de JMeter

1. Confirme todos los archivos de entrada extraídos en el repositorio de control de código fuente. Para ello, vaya a **Azure DevOps Portal**(<https://aex.dev.azure.com/>) y vaya al proyecto de DevOps **eShopOnWeb**.
1. Seleccione **Repositorios**. En la estructura de carpetas de código fuente, observa la subcarpeta **tests**. Observa los puntos suspensivos (...) y selecciona **Nuevo > Carpeta**.
1. especifica **jmeter** como el nombre de carpeta y **placeholder.txt** como nombre de archivo (Nota: no se puede crear una carpeta vacía).
1. Haz clic en **Confirmar** para confirmar la creación del archivo de marcador de posición y la carpeta jmeter.
1. En **Estructura de carpeta**, ve a la nueva subcarpeta **jmeter** creada. Haz clic en los **puntos suspensivos (...)** y selecciona **Cargar archivos**.
1. Con la opción **Examinar**, ve a la ubicación del archivo ZIP extraído y selecciona **config.yaml** y **quick_test.jmx**.
1. Haz clic en **Confirmar** para confirmar la carga de archivos en el control de código fuente.

#### Tarea 4: actualizar el archivo de definición de YAML del flujo de trabajo de CI/CD

1. Para crear y ejecutar una prueba de carga, la definición de flujo de trabajo de Azure Pipelines usa la extensión de **tarea de Azure Load Testing ** del marketplace de Azure DevOps. Abra la [extensión de tarea de Azure Load Testing](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting) en el marketplace de Azure DevOps y seleccione **Get it free** (Obtener gratis).
1. Seleccione la organización de Azure DevOps y, después, seleccione **Install** (Instalar) para instalar la extensión.
1. Desde el Portal de Azure DevOps y Project, ve a **Canalizaciones** y selecciona la canalización creada al principio de este ejercicio. Haga clic en **Editar**.
1. En el script YAML, ve a la **línea 56** y presiona ENTRAR/RETORNO para agregar una nueva línea vacía. (aparece justo antes de la fase de implementación del archivo YAML).
1. En la línea 57, selecciona el Asistente para tareas a la derecha y busca **Azure Load Testing**.
1. Completa el panel gráfico con la configuración correcta del escenario:
   - Suscripción de Azure: selecciona la suscripción que ejecuta los recursos de Azure
   - Archivo de prueba de carga: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
   - Grupo de recursos de prueba de carga: el grupo de recursos que tiene los recursos de Azure Load Testing
   - Nombre del recurso de prueba de carga: `eShopOnWebLoadTesting`
   - Nombre de ejecución de pruebas de carga: ado_run
   - Descripción de la ejecución de pruebas de carga: pruebas de carga desde ADO

1. Para confirmar la inserción de los parámetros como un fragmento de código de YAML, haz clic en **Agregar**.
1. Si la sangría del fragmento de código YAML proporciona errores (líneas onduladas rojas), corríjalas agregando dos espacios o pestañas para colocar el fragmento correctamente.  
1. En este fragmento de código de ejemplo, verás el aspecto que debería tener el código YAML

    ```yml
         - task: AzureLoadTest@1
          inputs:
            azureSubscription: 'AZURE DEMO SUBSCRIPTION'
            loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
            resourceGroup: 'az400m05l11-RG'
            loadTestResource: 'eShopOnWebLoadTesting'
            loadTestRunName: 'ado_run'
            loadTestRunDescription: 'load testing from ADO'
    ```

1. debajo del fragmento de código YAML insertado, agrega una nueva línea vacía presionando ENTRAR/RETORNO.
1. debajo de esta línea vacía, agrega un fragmento de código para la tarea Publicar, que muestra los resultados de la tarea de prueba de carga de Azure durante la ejecución de canalización:

    ```yml
        - publish: $(System.DefaultWorkingDirectory)/loadTest
          artifact: loadTestResults
    ```

1. Si la sangría del fragmento de código YAML proporciona errores (líneas onduladas rojas), corríjalas agregando dos espacios o pestañas para colocar el fragmento correctamente.  
1. Con ambos fragmentos de código agregados a la canalización de CI/CD, haz clic en **Guardar**.
1. Una vez que hayas guardados, haz clic en **Ejecutar** para desencadenar la canalización.
1. Confirma la rama (principal) y haz clic en el botón **Ejecutar** para iniciar la ejecución de la canalización.
1. En la página de estado de la canalización, haz clic en la fase **Compilación** para abrir los detalles de registro detallados de las distintas tareas de la canalización.
1. Espera a que la canalización inicie la fase de compilación hasta la tarea **AzureLoadTest** del flujo de canalización.
1. Mientras se ejecuta la tarea, ve a **Azure Load Testing** en Azure Portal y examina la forma en que la canalización crea un runTest nuevo, denominado **adoloadtest1**. Puedes seleccionarlo para ver los valores de resultado del trabajo TestRun.
1. Vuelve a la vista Ejecución de canalización de CI/CD de Azure DevOps, donde la tarea **AzureLoadTest** se ha completado correctamente. En los resultados del registro detallado, los valores resultantes de la prueba de carga también serán visibles:

    ```text
    Task         : Azure Load Testing
    Description  : Automate performance regression testing with Azure Load Testing
    Version      : 1.2.30
    Author       : Microsoft Corporation
    Help         : https://docs.microsoft.com/azure/load-testing/tutorial-cicd-azure-pipelines#azure-load-testing-task
    ==============================================================================
    Test '0d295119-12d0-482d-94be-a7b84787c004' already exists
    Uploaded test plan for the test
    Creating and running a testRun for the test
    View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%4b75-a1e0-27fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-787c004/testRunId/161046f1-d2d3-46f7-9d2b-c8a09478ce4c
    TestRun completed
    
    -------------------Summary ---------------
    TestRun start time: Mon Jul 24 2023 21:46:26 GMT+0000 (Coordinated Universal Time)
    TestRun end time: Mon Jul 24 2023 21:51:50 GMT+0000 (Coordinated Universal Time)
    Virtual Users: 50
    TestStatus: DONE
    
    ------------------Client-side metrics------------
    
    Homepage
    response time        : avg=1359ms min=59ms med=539ms max=16629ms p(90)=3127ms p(95)=5478ms p(99)=13878ms
    requests per sec     : avg=37
    total requests       : 4500
    total errors         : 0
    total error rate     : 0
    Finishing: AzureLoadTest
    
    ```

1. Has realizado una prueba de carga automatizada como parte de una ejecución de canalización. En la última tarea, especificarás las condiciones de error, lo que significa que no permitiremos que se inicie la fase de implementación si el rendimiento de la aplicación web está por debajo de un umbral determinado.

#### Tarea 5: agregar criterios de error y éxito a la canalización de pruebas de carga

En esta tarea, usarás criterios de error de prueba de carga para recibir alertas (ante un error en la ejecución de canalización como resultado) cuando la aplicación no cumpla con los requisitos de calidad.

1. En Azure DevOps, vaya al proyecto eShopOnWeb y abra **Repositorios**.
1. En Repos, ve a la subcarpeta **/tests/jmeter** creada y usada anteriormente.
1. Abre el archivo de prueba de carga *config.yaml**. Haz clic en **Editar** para permitir la edición del archivo.
1. Reemplace `failureCriteria: []` por el siguiente fragmento de código:

    ```text
    failureCriteria:
      - avg(response_time_ms) > 300
      - percentage(error) > 50
    ```

1. Guarda los cambios en config.yaml haciendo clic en **Confirmar** y confirma una vez más.
1. Vuelva a **Canalizaciones** y vuelva a ejecutar la canalización de **eShopOnWeb**. Después de unos minutos, completarás la ejecución con un estado de **error** para la tarea **AzureLoadTest**.
1. Abre la vista de registro detallado de la canalización y valida los detalles de **AzureLoadtest**. Este es un ejemplo del resultado:

    ```text
    Creating and running a testRun for the test
    View the load test run in progress at: https://portal.azure.com/#blade/Microsoft_Azure_CloudNativeTesting/NewReport//resourceId/%2fsubscriptions%2fb86d9ae1-7552-47fb2ea7f9f4%2fresourcegroups%2faz400m05l11-rg%2fproviders%2fmicrosoft.loadtestservice%2floadtests%2feshoponwebloadtesting/testId/0d295119-12d0-a7b84787c004/testRunId/f4bec76a-8b49-44ee-a388-12af34f0d4ec
    TestRun completed
    
    -------------------Summary ---------------
    TestRun start time: Mon Jul 24 2023 23:00:31 GMT+0000 (Coordinated Universal Time)
    TestRun end time: Mon Jul 24 2023 23:06:02 GMT+0000 (Coordinated Universal Time)
    Virtual Users: 50
    TestStatus: DONE
    
    -------------------Test Criteria ---------------
    Results          :1 Pass 1 Fail
    
    Criteria                     :Actual Value        Result
    avg(response_time_ms) > 300                       1355.29               FAILED
    percentage(error) > 50                                                  PASSED
    
    
    ------------------Client-side metrics------------
    
    Homepage
    response time        : avg=1355ms min=58ms med=666ms max=16524ms p(90)=2472ms p(95)=5819ms p(99)=13657ms
    requests per sec     : avg=37
    total requests       : 4531
    total errors         : 0
    total error rate     : 0
    ##[error]TestResult: FAILED
    Finishing: AzureLoadTest

    ```

1. La última línea de la salida de pruebas de carga indica **##[error]TestResult: FAILED**, dado que definimos un **FailCriteria** que tiene un tiempo medio de respuesta de >300 o que tiene un porcentaje de error de >20. Ahora se ve un tiempo de respuesta promedio que es superior a 300, y se marcará que se produjo un error en la tarea.

    > **Nota**: Ten en cuenta que en un escenario de la vida real, validarías el rendimiento de la App Service, y si el rendimiento está por debajo de un cierto umbral (por lo general significa que hay más carga en la aplicación web), podrías desencadenar una nueva implementación a un Azure App Service adicional. Como no podemos controlar el tiempo de respuesta de los entornos de laboratorio de Azure, decidimos revertir la lógica para garantizar el error.

1. El estado de ERROR de la tarea de canalización refleja realmente una validación exitosa de los criterios de requisitos de Azure Load Testing.

   > [!IMPORTANT]
   > Recuerda eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisar

En este ejercicio, has implementado una aplicación web para Azure App Service mediante Azure Pipelines, así como la implementación de un recurso de Azure Load Testing con TestRuns. A continuación, ha integrado el archivo config.yaml de pruebas de carga de JMeter en el control de código fuente de Azure Repos y amplía la canalización de CI/CD con Azure Load Testing. En el último ejercicio, has aprendido a definir los criterios de éxito de LoadTest.
