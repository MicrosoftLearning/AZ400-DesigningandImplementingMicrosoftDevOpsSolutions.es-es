---
lab:
  title: Supervisión del rendimiento de la aplicación con Azure Load Testing
  module: 'Module 09: Implement continuous feedback'
---

# Supervisión del rendimiento de aplicaciones con Application Insights y Azure Load Testing

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique una suscripción de Azure existente o cree una.

- Compruebe que tiene una cuenta Microsoft o una cuenta de Microsoft Entra con el rol Propietario en la suscripción de Azure y el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Introducción al laboratorio

**Azure Load Testing** es un servicio de prueba de carga totalmente administrado que permite generar una carga a gran escala. El servicio simula el tráfico de las aplicaciones, independientemente del lugar en que se hospeden. Los desarrolladores, evaluadores e ingenieros de control de calidad (QA) pueden usarlo para optimizar el rendimiento, la escalabilidad o la capacidad de las aplicaciones.
Cree rápidamente una prueba de carga para la aplicación web mediante una dirección URL y sin conocimientos previos de las herramientas de prueba. Azure Load Testing abstrae la complejidad y la infraestructura para ejecutar una prueba de carga a gran escala.
Para escenarios de prueba de carga más avanzados, puede crear una prueba de carga mediante la reutilización de un script de prueba existente de Apache JMeter, una popular herramienta de carga y rendimiento de código abierto. Por ejemplo, el plan de pruebas puede constar de varias solicitudes de aplicación, de llamadas a puntos de conexión no HTTP o del uso de parámetros y datos de entrada para que la prueba sea más dinámica.

En este laboratorio, obtendrás información sobre cómo puedes usar Azure Load Testing para simular pruebas de rendimiento en una aplicación web de ejecución en directo con diferentes escenarios de carga. Por último, aprenderás a integrar Azure Load Testing en las canalizaciones de CI/CD. 

## Objetivos

Después de completar este laboratorio, podrá:

- Implemente aplicaciones web de Azure App Service.
- Redactar y ejecutar una canalización de CI/CD basada en YAML.
- Implementar Azure Load Testing.
- Investigar el rendimiento de las aplicaciones web de Azure mediante Azure Load Testing.
- Integrar Azure Load Testing en las canalizaciones de CI/CD.

## Tiempo estimado: 60 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, que consta de un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de Azure DevOps **eShopOnWeb** que usarán varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asigna al proyecto el nombre **eShopOnWeb** y elige **Scrum** en la lista desplegable **Proceso de elemento de trabajo**. Haga clic en **Crear**.

    ![Crear proyecto](images/create-project.png)

#### Tarea 2: (omitir si ya la has completado) importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que usarán varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar**. En la ventana **Importar un repositorio de Git** pega la siguiente dirección URL https://github.com/MicrosoftLearning/eShopOnWeb.git y haz clic en **Importar**:

    ![Importar repositorio](images/import-repo.png)

2. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **.azure** contiene la infraestructura de Bicep&ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - La carpeta **.github** contiene definiciones de flujo de trabajo de GitHub YAML.
    - La carpeta **src** contiene el sitio web .NET 7 que se usa en los escenarios de laboratorio.

#### Tarea 2: Crear recursos de Azure

En esta tarea, crearás una aplicación web de Azure mediante Cloud Shell en Azure Portal.

1. En el equipo del laboratorio, inicia un explorador web, ve a [**Azure Portal**](https://portal.azure.com) e inicia sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción de Azure que vas a usar en este laboratorio, así como el rol Administrador global en el inquilino de Microsoft Entra asociado a esta suscripción.
2. En Azure Portal, en la barra de herramientas, haz clic en el icono **Cloud Shell** situado directamente a la derecha del cuadro de texto de búsqueda.
3. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.
    >**Nota**: si es la primera vez que inicias **Cloud Shell** y aparece el mensaje **No tiene ningún almacenamiento montado**, selecciona la suscripción que usas en este laboratorio y haz clic en **Crear almacenamiento**.

4. En el símbolo del sistema de **Bash**, en el panel de **Cloud Shell**, ejecuta el siguiente comando para crear un grupo de recursos (reemplaza el marcador de posición `<region>` por el nombre de la región de Azure más cercana, como "eastus").

    ```bash
    RESOURCEGROUPNAME='az400m09l16-RG'
    LOCATION='<region>'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

5. Para crear un plan de App Service ejecuta el comando siguiente:

    ```bash
    SERVICEPLANNAME='az400l16-sp'
    az appservice plan create --resource-group $RESOURCEGROUPNAME \
        --name $SERVICEPLANNAME --sku B3 
    ```

6. Crea una nueva aplicación web con un nombre único.

    ```bash
    WEBAPPNAME=partsunlimited$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME 
    ```

    > **Nota**: registra el nombre de la aplicación web. Lo necesitará más adelante en este laboratorio.

### Ejercicio 1: configuración de canalizaciones de CI/CD como código con YAML en Azure DevOps

En este ejercicio, configurarás canalizaciones de CI/CD como código con YAML en Azure DevOps.

#### Tarea 1: agregar una definición de compilación e implementación de YAML

En esta tarea, agregarás una definición de compilación de YAML al proyecto existente.

1. Vuelve al panel **Canalizaciones** del centro de **Canalizaciones**.
2. En la ventana **Crear la primera canalización**, haz clic en **Crear canalización**.

    > **Nota**: usaremos el asistente para crear una nueva definición de canalización de YAML basada en nuestro proyecto.

3. En el panel **¿Dónde está el código?**, haz clic en la opción **Azure Repos Git (YAML)**.
4. En el panel **Seleccionar un repositorio**, haz clic en **eShopOnWeb**.
5. En la pantalla **Configuración de la canalización**, desplázate hacia abajo y selecciona **Canalización inicial**.
6. **Selecciona** todas las líneas de la canalización inicial y elimínalas.
7. **Copia** la canalización de plantilla completa de la siguiente manera, sabiendo que deberás realizar modificaciones de parámetros **antes de guardar** los cambios:

```
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
4. Establece el cursor en una nueva línea al final de la definición de YAML (línea 69).

    > **Nota**: esta será la ubicación donde se agregan nuevas tareas.

5. En la lista de tareas del lado derecho del panel de código, busca y selecciona la tarea **Implementación de Azure App Service**.
6. En el panel **Implementación de Azure App Service**, especifica la siguiente configuración y haz clic en **Agregar**:

    - En la lista desplegable de **Suscripción de Azure**, selecciona la suscripción de Azure en la que has implementado los recursos de Azure anteriormente en el laboratorio, haz clic en **Autorizar** y, cuando se te solicite, autentícate mediante la misma cuenta de usuario que has usado durante la implementación de recursos de Azure.
    - En la lista desplegable **Nombre de App Service** selecciona el nombre de la aplicación web que has implementado anteriormente en el laboratorio.
    - En el cuadro de texto **Paquete o carpeta**, **actualiza** el valor predeterminado a `$(Build.ArtifactStagingDirectory)/**/Web.zip`.
7. Para confirmar la configuración del panel Asistente, haz clic en el botón **Agregar**.

    > **Nota**: esto agregará automáticamente la tarea de implementación a la definición de canalización de YAML.

8. El fragmento de código agregado al editor debe ser similar a lo siguiente, lo que refleja el nombre de los parámetros azureSubscription y WebappName:

> **Nota**: el parámetro **packageForLinux** es engañoso en el contexto de este laboratorio, pero es válido para Windows o Linux.

    ```yaml
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
    ```
9. Haz clic en **Guardar**, en el panel **Guardar**, y haz clic en **Guardar** de nuevo para confirmar el cambio directamente en la rama maestra.

    > **Nota**: dado que el CI-YAML original no se ha configurado para desencadenar automáticamente una nueva compilación, tenemos que iniciarla manualmente.

10. En Azure DevOps del menú izquierdo, ve a la pestaña **Canalizaciones** y selecciona **Canalizaciones** de nuevo.
11. Abre la canalización **EShopOnWeb_MultiStageYAML** y haz clic en **Ejecutar canalización**.
12. Confirma **Ejecutar** desde el panel que aparece.
13. Observa que aparecen las dos fases diferentes: **Compilar una solución .Net Core** e **Implementación de una aplicación web en Azure**.
14. Espera a que se inicie la canalización y a que finalice correctamente la fase de compilación.
15. Una vez que la fase de implementación quiere iniciarse, se te pedirán **los permisos necesarios**, así como una barra naranja que indica:

    ```text
    This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
    ```

16. Haz clic en **Vista**.
17. En el panel **Esperando revisión**, haz clic en **Permitir**.
18. Valida el mensaje en la ventana emergente **Permitir** y confirma haciendo clic en **Permitir**.
19. Esto activa la fase de implementación. Espera a que la implementación se complete correctamente.

#### Tarea 2: revisar el sitio implementado

1. Vuelve a la ventana del explorador web que muestra Azure Portal y ve a la hoja que muestra las propiedades de la aplicación web de Azure
2. En la hoja Aplicación web de Azure, haz clic en **Información general** y, en la hoja de información general, haz clic en **Examinar** para abrir el sitio en una nueva pestaña del explorador web.
3. Comprueba que el sitio implementado se cargue según lo previsto en la nueva pestaña del explorador, que muestra el sitio web de comercio electrónico EShopOnWeb.

### Ejercicio 2: implementación y configuración de Azure Load Testing

En este ejercicio, implementarás un recurso de Azure Load Testing en Azure y configurarás diferentes escenarios de pruebas de carga para Azure App Service en directo.

#### Tarea 1: implementar pruebas de carga de Azure

En esta tarea, implementarás un recurso de Azure Load Testing en la suscripción de Azure.

1. Desde Azure Portal (https://portal.azure.com), ve a **Crear recurso de Azure**.
2. En el campo de búsqueda "Buscar en servicios y marketplace", escribe **Azure Load Testing**.
3. En los resultados de la búsqueda, selecciona **Azure Load Testing** (publicado por Microsoft).
4. En la página de Azure Load Testing, haz clic en **Crear** para iniciar el proceso de implementación.
5. En la página "Creación de un recurso de Load Testing", da los detalles necesarios para la implementación del recurso:
- **Suscripción**: selecciona la suscripción de Azure
- **Grupo de recursos**: selecciona el grupo de recursos que has usado para implementar Web App Service en el ejercicio anterior.
- **Nombre**: EShopOnWebLoadTesting
- **Región**: selecciona una región cercana a tu área

    > **Nota**: el servicio Azure Load Testing no está disponible en todas las regiones de Azure.

6. Para validar tu configuración, haz clic en **Revisar y crear**.
7. Haz clic en **Crear** para confirmar y obtener el recurso de Azure Load Testing implementado.
8. Has cambiado a la página "La implementación está en curso". La implementación tarda unos minutos en finalizar.
9. Haz clic en **Ir al recurso** en la página de progreso de la implementación para ir al recurso de Azure Load Testing **EShopOnWebLoadTesting**. 

    > **Nota**: si has cerrado la hoja o has cerrado Azure Portal durante la implementación del recurso de Azure Load Testing, puedes encontrarlo de nuevo en el campo de búsqueda de Azure Portal o desde Recursos/Lista reciente de recursos. 

#### Tarea 2: crear pruebas de Azure Load Testing

En esta tarea, crearás distintas pruebas de Azure Load Testing con diferentes opciones de configuración de carga. 

10. Desde la hoja Recurso de Azure Load Testing de **EShopOnWebLoadTesting**, observa la **introducción a una prueba rápida** y haz clic en el botón **Prueba rápida**.
11. Completa los parámetros y la configuración siguientes para crear una prueba de carga:
- **Dirección URL de prueba**: escribe la dirección URL de Azure App Service que has implementado en el ejercicio anterior (EShopOnWeb... azurewebsites.net), **incluida la https://**
- **Especificar carga**: usuarios virtuales
- **Número de usuarios virtuales**: 50
- **Duración de la prueba (segundos):** 120
- **Tiempo de aumento (segundos)**: 0
12. Para confirmar la configuración de la prueba, haz clic en **Ejecutar prueba**.
13. La prueba durará unos 2 minutos. 
14. Con la prueba en ejecución, vuelve a la página del recurso de Azure Load Testing **EShopOnWebLoadTesting** Azure Load Testing y ve a **Pruebas**, selecciona **Pruebas** y visualiza una prueba **Get_eshoponweb...**
15. En el menú superior, haz clic en **Crear**, **Crear una prueba rápida** para crear una segunda prueba de carga.
16. Completa los siguientes parámetros y ajustes para crear otra prueba de carga:
- **Dirección URL de prueba**: escribe la dirección URL de Azure App Service que has implementado en el ejercicio anterior (EShopOnWeb... azurewebsites.net), **incluida la https://**
- **Especificar carga**: solicitudes por segundo (RPS)
- **Solicitudes por segundo (RPS)**: 100
- **Tiempo de respuesta (milisegundos)**: 500
- **Duración de la prueba (segundos)**: 120
- **Tiempo de aumento (segundos)**: 0
17. Para confirmar la configuración de la prueba, haz clic en **Ejecutar prueba**.
18. La prueba durará unos 2 minutos.

#### Tarea 3: validar los resultados de Azure Load Testing

En esta tarea, validarás el resultado de una prueba de ejecución de Azure Load Testing. 

Una vez completadas ambas pruebas rápidas, vamos a realizar algunos cambios en ellas y validar los resultados.

19. En la hoja Recurso de **EShopOnWebLoadTesting**, ve a **Pruebas** y selecciona la primera prueba Get_eshoponwebyaml. Haga clic en **Editar** en el menú superior.
20. Aquí, el portal te permite cambiar el **nombre de la prueba** del valor predeterminado generado a uno más descriptivo. También te permite hacer cambios en cualquiera de los parámetros definidos anteriormente.
21. En la hoja **Editar prueba**, ve a la pestaña **Plan de prueba**. 
22. Aquí es donde puedes administrar el archivo de script de prueba de carga de **Apache JMeter**, que es lo que Azure Load Testing usa como marco. Observa el archivo **quick_test.jmx**. Selecciona este archivo para **abrirlo** en la máquina virtual del laboratorio. En la ventana emergente, selecciona **Visual Studio Code** como editor para abrir el archivo.
23. Observa la estructura del lenguaje XML del archivo.

    > Nota: para obtener información adicional y entender la sintaxis más avanzada de Apache JMeter, consulta el siguiente vínculo de [Azure Load Testing - Jmeter](https://learn.microsoft.com/en-us/azure/load-testing/how-to-create-and-run-load-test-with-jmeter-script).

24. De nuevo en la vista **Pruebas**, en la que se muestran ambas pruebas, selecciona cualquiera de ellas para abrir una vista más detallada, con solo un **clic**. Esto redirige a la página de prueba más detallada. Desde aquí, puedes validar los detalles de las ejecuciones reales seleccionando **TestRun_mm/dd/yy-hh:hh** de la lista que aparece.
25. En la página **TestRun** detallada, identifica el resultado real de la simulación de Azure Load Testing. Algunos de los valores son:
- Carga / Total de solicitudes
- Duration
- Tiempo de respuesta (Muestra el resultado en segundos, lo que refleja el tiempo de respuesta del percentil 90. Esto significa que, para el 90% de las solicitudes, el tiempo de respuesta estará dentro de los resultados especificados).
- Rendimiento en solicitudes por segundo
26. A continuación, se representan muchos de estos valores en líneas gráficas del panel y gráficos.
27. Dedica unos minutos a **comparar los resultados** de las pruebas simuladas entre sí e **identificar el impacto** de más usuarios sobre el rendimiento de App Service.

### Ejercicio 2: automatización de una prueba de carga con CI/CD en canalizaciones de Azure DevOps

Empiece a automatizar las pruebas de carga en Azure Load Testing agregándolo a una canalización de CI/CD. Después de ejecutar una prueba de carga en Azure Portal, exporta los archivos de configuración y configura una canalización de CI/CD en Azure Pipelines (Acciones de GitHub tiene una función similar).

Después de completar este ejercicio, tendrás un flujo de trabajo de CI/CD configurado para ejecutar una prueba de carga con Azure Load Testing.

#### Tarea 1: identificar los detalles de la conexión al servicio de ADO

En esta tarea, concederás los permisos necesarios a la entidad de servicio de Azure DevOps Service Connection.

1. En **Azure DevOps Portal** (https://dev.azure.com), ve al proyecto **EShopOnWeb**.
2. Selecciona **Configuración del proyecto** en la esquina inferior izquierda.
3. En la sección **Canalizaciones**, selecciona **Conexiones de servicio**.
4. Observa la conexión de servicio, que tiene el nombre de la suscripción de Azure que has usado para implementar recursos de Azure al principio del ejercicio del laboratorio.
5. **Selecciona Service Connection**. En la pestaña **Información general**, ve a **Detalles** y selecciona **Administrar la entidad de servicio**.
6. Esto te redirige a Azure Portal, desde donde abre los detalles de la **Entidad de servicio** correspondientes al objeto de identidad.
7. Copia el valor **Nombre para mostrar** (con el formato Nombre_de_Organización_ADO_EShopOnWeb_-b86d9ae1-7552-4b75-a1e0-27fb2ea7f9f4), porque lo necesitarás en los pasos siguientes.

#### Tarea 2: conceder permisos a la entidad de servicio

Azure Load Testing usa RBAC de Azure para conceder permisos para realizar actividades específicas en el recurso de prueba de carga. Para ejecutar una prueba de carga desde la canalización de CI/CD, concede el rol **Colaborador de la prueba de carga** a la entidad de servicio.

1. En **Azure Portal**, ve al recurso de **Azure Load Testing**.
2. Selecciona **Control de acceso (IAM)** > Agregar > Agregar asignación de roles.
3. En la pestaña **Rol**, seleccione **Colaborador de pruebas de carga** en la lista de roles de funciones de trabajo.
4. En la pestaña **Miembros**, selecciona **Seleccionar miembros** y luego usa el **nombre para mostrar** que has copiado anteriormente para buscar en la entidad de servicio.
5. Selecciona la **entidad de servicio** y luego, **Seleccionar**.
6. En la **pestaña Revisar + asignar**, seleccione **Revisar + asignar** para agregar la asignación de roles.

Ahora es posible usar la conexión de servicio en la definición de flujo de trabajo de Azure Pipelines para acceder al recurso de prueba de carga de Azure.

#### Tarea 3: exportar archivos de entrada de prueba de carga e importar al control de código fuente de Azure DevOps

Para ejecutar una prueba de carga con Azure Load Testing en un flujo de trabajo de CI/CD, es necesario agregar la configuración de prueba de carga y los archivos de entrada del repositorio de control de código fuente. Si tiene una prueba de carga existente, puede descargar los valores de configuración y todos los archivos de entrada desde Azure Portal.

Realice los pasos siguientes para descargar los archivos de entrada de una prueba de carga existente en Azure Portal:

1. En **Azure Portal**, ve al recurso de **Azure Load Testing**.
2. En el panel izquierdo, selecciona **Pruebas** para ver la lista de pruebas y selecciona **tu prueba**.
3. Seleccione los puntos suspensivos (**...**) que hay junto a la serie de pruebas con la que trabaje y, después, seleccione **Descargar archivo de entrada**.
4. El explorador descarga una carpeta comprimida que contiene los archivos de entrada de pruebas de carga.
5. Use cualquier herramienta zip para extraer los archivos de entrada. La carpeta contiene los archivos siguientes:

- *config.yaml*: el archivo de configuración de YAML de prueba de carga. Haga referencia a este archivo en la definición de flujo de trabajo de CI/CD.
- *quick_test.jmx*: el script de prueba de JMeter

6. Confirme todos los archivos de entrada extraídos en el repositorio de control de código fuente. Para ello, ve al **Portal de Azure DevOps(https://dev.azure.com)) y ve al proyecto de DevOps **EShopOnWeb****. 
7. Seleccione **Repositorios**. En la estructura de carpetas de código fuente, observa la subcarpeta **pruebas**. Observa los puntos suspensivos (...) y selecciona **Nuevo > Carpeta**.
8. especifica **jmeter** como nombre de carpeta y **placeholder.txt** para el nombre de archivo (Nota: no se puede crear una carpeta como vacía)
9. Haz clic en **Confirmar** para confirmar la creación del archivo de marcador de posición y la carpeta jmeter.
10. En la **Estructura de carpetas**, ve a la subcarpeta **jmeter** recién creada. Haz clic en los **puntos suspensivos (...)** y selecciona **Cargar archivos**.
11. Con la opción **Examinar**, ve a la ubicación del archivo ZIP extraído y selecciona **config.yaml** y **quick_test.jmx**.
12. Haz clic en **Confirmar** para confirmar la carga de archivos en el control de código fuente.

#### Tarea 4: actualizar el archivo de definición de YAML del flujo de trabajo de CI/CD

En esta tarea, importarás la extensión Azure Load Testing - Azure DevOps Marketplace y la actualización de la canalización de CI/CD existente con la tarea AzureLoadTest.

1. Para crear y ejecutar una prueba de carga, la definición de flujo de trabajo de Azure Pipelines usa la extensión de **tarea de Azure Load Testing ** del marketplace de Azure DevOps. Abra la [extensión de tarea de Azure Load Testing](https://marketplace.visualstudio.com/items?itemName=AzloadTest.AzloadTesting) en el marketplace de Azure DevOps y seleccione **Get it free** (Obtener gratis).
2. Seleccione la organización de Azure DevOps y, después, seleccione **Install** (Instalar) para instalar la extensión.
3. Desde el Portal de Azure DevOps y Project, ve a **Canalizaciones** y selecciona la canalización creada al principio de este ejercicio. Haga clic en **Editar**.
4. En el script YAML, ve a **la línea 56** y presiona ENTRAR/RETORNO para agregar una nueva línea vacía. (aparece justo antes de la fase de implementación del archivo YAML).
5. En la línea 57, selecciona el Asistente para tareas en el lado derecho y busca **Azure Load Testing**.
6. Completa el panel gráfico con la configuración correcta del escenario:
- Suscripción de Azure: selecciona la suscripción que ejecuta los recursos de Azure
- Archivo de prueba de carga: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml' 
- Grupo de recursos de prueba de carga: el grupo de recursos que tiene los recursos de Azure Load Testing
- Nombre del recurso de prueba de carga: ESHopOnWebLoadTesting
- Nombre de ejecución de pruebas de carga: ado_run
- Descripción de la ejecución de pruebas de carga: pruebas de carga desde ADO
7. Para confirmar la inserción de los parámetros como un fragmento de código de YAML, haz clic en **Agregar**
8. Si la sangría del fragmento de código YAML presenta errores (líneas onduladas rojas), corrígelas agregando 2 espacios o tabulaciones para colocar el fragmento correctamente.  
9. En este fragmento de código de ejemplo, verás el aspecto que debería tener el código YAML
```
     - task: AzureLoadTest@1
      inputs:
        azureSubscription: 'AZURE DEMO SUBSCRIPTION(b86d9ae1-1234-4b75-a8e7-27fb2ea7f9f4)'
        loadTestConfigFile: '$(Build.SourcesDirectory)/tests/jmeter/config.yaml'
        resourceGroup: 'az400m05l11-RG'
        loadTestResource: 'EShopOnWebLoadTesting'
        loadTestRunName: 'ado_run'
        loadTestRunDescription: 'load testing from ADO'
```
10. debajo del fragmento de código YAML insertado, agrega una nueva línea vacía presionando ENTRAR/RETORNO. 
11. debajo de esta línea vacía, agrega un fragmento de código para la tarea Publicar, que muestra los resultados de la tarea de prueba de carga de Azure durante la ejecución de canalización:

```
    - publish: $(System.DefaultWorkingDirectory)/loadTest
      artifact: loadTestResults
```
12.  Si la sangría del fragmento de código YAML presenta errores (líneas onduladas rojas), corrígelas agregando 2 espacios o tabulaciones para colocar el fragmento correctamente.  
13. Con ambos fragmentos de código agregados a la canalización de CI/CD, **guarda** los cambios. 
14. Una vez guardada, haz clic en **Ejecutar** para desencadenar la canalización.
15. Confirma la rama (main) y haz clic en el botón **Ejecutar** para iniciar la ejecución de la canalización.
16. En la página de estado de la canalización, haz clic en la fase **Compilación** para abrir los detalles de registro detallados de las distintas tareas de la canalización.
17. Espera a que la canalización inicie la fase de compilación y llegue a la tarea **AzureLoadTest** en el flujo de la canalización. 
18. Mientras se ejecuta la tarea, ve a **Azure Load Testing** en Azure Portal y ve cómo la canalización crea un RunTest nuevo, denominado **adoloadtest1**. Puedes seleccionarlo para ver los valores de resultado del trabajo TestRun.
19. Vuelve a la vista Ejecución de canalización de CI/CD de Azure DevOps, donde la **tarea AzureLoadTest** se ha completado correctamente. En los resultados del registro detallado, los valores resultantes de la prueba de carga también serán visibles:

```
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
20. Has realizado una prueba de carga automatizada como parte de una ejecución de canalización. En la última tarea, especificarás las condiciones de error, lo que significa que no permitiremos que se inicie la fase de implementación si el rendimiento de la aplicación web está por debajo de un umbral determinado. 

#### Tarea 5: agregar criterios de error y éxito a la canalización de pruebas de carga

En esta tarea, usarás criterios de error de prueba de carga para recibir alertas (ante un error en la ejecución de canalización como resultado) cuando la aplicación no cumpla con los requisitos de calidad.

1. En Azure DevOps, ve al proyecto EShopOnWeb y abre **Repos**.
2. En Repos, ve a la subcarpeta **/tests/jmeter** creada y usada anteriormente.
3. Abre el archivo de prueba de carga *config.yaml**. Haz clic en **Editar** para permitir la edición del archivo.
4. Agrega el siguiente fragmento de código al final del archivo:

```
failureCriteria:
  - avg(response_time_ms) > 300
  - percentage(error) > 50
```
5. Guarda los cambios en config.yaml haciendo clic en **Confirmar** y confirma una vez más.
6. Vuelve a **Canalizaciones** y vuelve a ejecutar la canalización **EShopOnWeb**. Después de unos minutos, completará la ejecución con un estado **error** para la tarea **AzureLoadTest**. 
7. Abre la vista de registro detallado de la canalización y valida los detalles de **AzureLoadtest**. Este es un ejemplo similar del resultado:

```
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

8. La última línea de la salida de pruebas de carga indica **##[error]TestResult: FAILED**, dado que definimos un **FailCriteria** que tiene un tiempo medio de respuesta de >300 o que tiene un porcentaje de error de >20. Ahora se ve un tiempo de respuesta promedio que es superior a 300, y se marcará que se produjo un error en la tarea. 

    > Nota: imagina un escenario real en el que validarías el rendimiento de App Service. Si el rendimiento está por debajo de una retención determinada, significa que hay más carga en la aplicación web, lo que podría desencadenar una nueva implementación en un Azure App Service adicional. Como no podemos controlar el tiempo de respuesta de los entornos de laboratorio de Azure, decidimos revertir la lógica para garantizar el error.

9.  El estado FAILED de la tarea de canalización refleja realmente una validación exitosa de los criterios de requisitos de Azure Load Testing.

### Ejercicio 3: eliminación de los recursos del laboratorio de Azure

En este ejercicio, quitarás los recursos de Azure aprovisionados en este laboratorio para eliminar cargos inesperados.

>**Nota**: No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

#### Tarea 1: eliminar los recursos del laboratorio de Azure

En esta tarea, usarás Azure Cloud Shell para quitar los recursos de Azure aprovisionados en este laboratorio con el propósito de eliminar cargos innecesarios.

1. En Azure Portal, abra la sesión de shell de **Bash** en el panel **Cloud Shell**.
2. Ejecute el comando siguiente para enumerar todos los grupos de recursos que se han creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].name" --output tsv
    ```

3. Ejecute el comando siguiente para eliminar todos los grupos de recursos que ha creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro --nowait). Aunque podrá ejecutar otro comando de la CLI de Azure inmediatamente después en la misma sesión de Bash, los grupos de recursos tardarán unos minutos en quitarse.

## Revisar

En este ejercicio, has implementado una aplicación web para Azure App Service mediante Azure Pipelines, así como la implementación de un recurso de Azure Load Testing con TestRuns. A continuación, has integrado el archivo config.yaml de pruebas de carga de Jmeter en el control de código fuente de Azure DevOps Repos y has ampliado la canalización de CI/CD con Azure Load Testing. En el último ejercicio, has aprendido a definir los criterios de éxito de LoadTest.
