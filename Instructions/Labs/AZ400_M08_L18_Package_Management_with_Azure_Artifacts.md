---
lab:
  title: Administración de paquetes con Azure Artifacts
  module: 'Module 08: Design and implement a dependency management strategy'
---

# Administración de paquetes con Azure Artifacts

# Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurar una organización de Azure DevOps:**: si aún no tienes una organización Azure DevOps que puedas usar para este laboratorio, crea una siguiendo las instrucciones disponibles en [Requisitos previos para el laboratorio AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configurar el proyecto de ejemplo de EShopOnWeb:** si aún no tienes el proyecto EShopOnWeb de ejemplo que puedes usar para este laboratorio, crea uno siguiendo las instrucciones disponibles en [Requisitos previos para el laboratorio AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- Visual Studio 2022 Community Edition está disponible en la [página de descargas de Visual Studio](https://visualstudio.microsoft.com/downloads/). La instalación de Visual Studio 2022 debe incluir **Desarrollo ASP.<nolink>NET y web, ** **desarrollo Azure**, y **desarrollo multiplataforma de .NET Core**.

## Introducción al laboratorio

Azure Artifacts facilita la detección, la instalación y la publicación de paquetes NuGet, npm y Maven en Azure DevOps. Está profundamente integrado con otras características de Azure DevOps, como la compilación, lo que hace que la administración de paquetes sea una parte perfecta de los flujos de trabajo existentes.

## Objetivos

Después de completar este laboratorio, podrá:

- Cree una fuente y conéctese a ella.
- Cree y publique un paquete de NuGet.
- Importe un paquete de NuGet.
- Actualice un paquete de NuGet.

## Tiempo estimado: 40 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, queremos recordarte que debes validar los requisitos previos del laboratorio, que tengas lista una organización de Azure DevOps y que hayas creado el proyecto EShopOnWeb. Consulta las instrucciones para obtener más detalles.

#### Tarea 1: configurar la solución EShopOnWeb en Visual Studio

En esta tarea, configurarás Visual Studio para prepararte para el laboratorio.

1. Asegúrate de que estás viendo el proyecto del equipo **EShopOnWeb** en el portal de Azure DevOps.

    > **Nota**: para acceder directamente a la página del proyecto, ve a la dirección URL [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb), donde el marcador de posición `<your-Azure-DevOps-account-name>` representa el nombre de la organización de Azure DevOps.

1. En el menú vertical del lado izquierdo del panel **EShopOnWeb**, haz clic en **Repos**.
1. En el panel **Archivos**, haz clic en **Clonar**, selecciona la flecha desplegable situada junto a **Clonar en VS Code** y, en el menú desplegable, selecciona **Visual Studio**.
1. Si el sistema pregunta si deseas continuar, haz clic en **Abrir**.
1. Inicia sesión con la cuenta de usuario que planeas usar en tu organización de Azure DevOps.
1. En la interfaz de Visual Studio, en la ventana emergente de **Azure DevOps**, acepta la ruta de acceso local predeterminada (C:\EShopOnWeb) y haz clic en **Clonar**. Esto importará el proyecto automáticamente en Visual Studio.
1. Deja abierta la ventana de Visual Studio para usarla en el laboratorio.

### Ejercicio 1: trabajo con Azure Artifacts

En este ejercicio, aprenderás a trabajar con Azure Artifacts siguiendo estos pasos:

- crea una fuente y conéctate a ella.
- crea y publica un paquete de NuGet.
- importa un paquete de NuGet.
- actualiza un paquete de NuGet.

#### Tarea 1: creación y conexión a una fuente

En esta tarea, crearás una fuente y te conectarás a ella.

1. En la ventana del explorador web que muestra la configuración del proyecto en el portal de Azure DevOps, en el panel de navegación vertical, selecciona **Artefactos**.
1. Con el centro de **Artefactos** en pantalla, haz clic en **+ Crear fuente** en la parte superior del panel.

    > **Nota**: esta fuente será una colección de paquetes NuGet disponibles para los usuarios de la organización y se almacenará con con la fuente de NuGet pública como elemento del mismo nivel. El escenario de este laboratorio se centrará en el flujo de trabajo para usar Azure Artifacts, por lo que las decisiones de arquitectura y desarrollo reales son meramente ilustrativas.  Esta fuente incluirá funciones comunes que se pueden compartir entre proyectos de esta organización.

1. En el panel **Crear nueva fuente**, en el cuadro de texto **Nombre**, escribe **EShopOnWebShared**. En la sección **Ámbito**, selecciona la opción **Organización**, deja otras opciones con sus valores predeterminados y haz clic en **Crear**.

    > **Nota**: cualquier usuario que quiera conectarse a esta fuente de NuGet debe configurar su entorno.

1. Cuando vuelvas al centro de **Artefactos**, haz clic en **Conectar a la fuente**.
1. En el panel **Conectar para la fuente**, en la sección **NuGet**, selecciona **Visual Studio** y, en el panel de **Visual Studio**, copia la dirección URL de** origen**. (https://pkgs.dev.azure.com/<Azure-DevOps-Org-Name>_packaging/EShopOnWebShared/nuget/v3/index.json)
1. Vuelve a la ventana de **Visual Studio**.
1. En la ventana de Visual Studio, haz clic en el menú **Herramientas**, en el menú desplegable, selecciona **Administrador de paquetes NuGet** y, en el menú en cascada, **Configuración del administrador de paquetes**.
1. En el cuadro de diálogo **Opciones**, haz clic en **Orígenes de paquete** de paquetes y haz clic en el signo de más para agregar un nuevo origen de paquete.
1. En la parte inferior del cuadro de diálogo, en el cuadro de texto **Nombre**, reemplaza **Origen del paquete** por **EShopOnWebShared** y, en el cuadro de texto **Origen**, pega la dirección URL que copiaste en el portal de Azure DevOps.
1. Haz clic en ** Actualizar** y después, en **Aceptar** para finalizar la incorporación.

    > **Nota**: Visual Studio ahora está conectado a la nueva fuente.

#### Tarea 2: Creación y publicación de un paquete NuGet desarrollado de forma interna

En esta tarea, crearás y publicarás un paquete NuGet personalizado desarrollado de forma interna.

1. En la ventana de Visual Studio que usaste para configurar el nuevo origen del paquete, en el menú principal, haz clic en **Archivo**, en el menú desplegable, haz clic en **Nuevo** y después en el menú en cascada, haz clic en **Proyecto**.

    > **Nota**: Ahora crearemos un ensamblado compartido que se publicará como un paquete NuGet para que otros equipos puedan integrarlo y mantenerse al día sin tener que trabajar directamente con el origen del proyecto.

1. En la página **Plantillas de proyecto recientes** del panel **Crear un nuevo proyecto**, usa el cuadro de texto de búsqueda para buscar la plantilla **Biblioteca de clases**, selecciona la plantilla de C# y haz clic en **Siguiente**.
1. En la página **Biblioteca de clases** del panel **Crear un nuevo proyecto**, especifica la siguiente configuración y haz clic en **Crear**:

    | Configuración | Valor |
    | --- | --- |
    | Nombre de proyecto | **EShopOnWeb.Shared** |
    | Location | acepte el valor predeterminado |
    | Solución | **Crear nueva solución** |
    | Nombre de la solución | **EShopOnWeb.Shared** |
    
    Activa la casilla **Colocar la solución y el proyecto en el mismo directorio**. 

1. Haga clic en Next. Acepta **.NET 6.0 (compatibilidad a largo plazo)** como opción de Framework.
1. Confirma la creación del proyecto pulsando el botón **Crear**.
    
1. En la interfaz de Visual Studio, en el panel **Explorador de soluciones**, haz clic con el botón derecho en **Class1.cs**, en el menú contextual, selecciona **Eliminar** y, cuando se te pida confirmación, haz clic en **Aceptar**.
1. Pulsa **Ctrl+Shift+B** o **haz clic con el botón derecho en el proyecto** EShopOnWeb.Shared y selecciona **Compilar** para compilar el proyecto.

    > **Nota**: En la siguiente tarea usaremos **NuGet.exe** para generar un paquete NuGet directamente desde el proyecto compilado, pero requiere que el proyecto se cree primero.

1. Vuelve a la pestaña del explorador web en la que aparece el portal de Azure DevOps.
1. Ve al panel de **Conectar a la fuente**, en la sección **NuGet** y selecciona **NuGet.exe**. Se mostrará el panel **panel NuGet.exe**.
1. En el panel **NuGet.exe**, haz clic en **Obtener las herramientas**.
1. En el panel **Obtener las herramientas**, haz clic en el enlace **Descargar la versión más reciente de NuGet**. Se abrirá automáticamente otra pestaña del explorador que muestra la página **Versiones de distribución de NuGet disponibles**.
1. En la página **Versiones de distribución** de NuGet disponibles, selecciona **nuget.exe: se recomienda la versión 6.x** más reciente y descargar el archivo ejecutable en la carpeta local **EShopOnWeb.Shared Project** (si mantienes las ubicaciones de carpeta predeterminadas, debe ser C:\EShopOnWeb\EShopOnWeb.Shared).
1. Selecciona el **archivo nuget.exe** y abre tus propiedades haciendo clic con el botón derecho en el archivo y seleccionando **Propiedades** en el menú contextual.
1. En la ventana contexto Propiedades, en la pestaña **General**, selecciona **Desbloquear** en la sección Seguridad. Para confirmar, pulsa **Aplicar** y **Aceptar**.
1. En tu estación de trabajo de laboratorio, abre el menú Inicio y busca **Windows PowerShell**. A continuación, en el menú en cascada, haz clic en **Abrir Windows PowerShell como administrador**.
1. En la ventana **Administración: Windows PowerShell**, ve a la carpeta EShopOnWeb.Shared ejecutando el siguiente comando: 

```
cd c:\EShopOnWeb\EShopOnWeb.Shared
```
ejecuta lo siguiente para crear un archivo **.nupkg** desde el proyecto.

    > **Note**: This is a shortcut to package the NuGet bits for deployment. NuGet is highly customizable. To learn more, refer to the [NuGet package creation page](https://docs.microsoft.com/en-us/nuget/create-packages/overview-and-workflow).

    ```
    .\nuget.exe pack ./EShopOnWeb.Shared.csproj
    ```

    > **Note**: Disregard any warnings displayed in the **Administrator: Windows PowerShell** window.

    > **Note**: NuGet builds a minimal package based on the information it is able to identify from the project. For example, note that the name is **EShopOnWeb.Shared.1.0.0.nupkg**. That version number was retrieved from the assembly.

1. Después de crear el paquete correctamente, ejecuta lo siguiente para publicar el paquete en la fuente **EShopOnWebShared**:

    > **Nota**: Debes proporcionar una **clave de API**, que puede ser cualquier cadena no vacía. Estamos usando **AzDO** aquí. Cuando se te solicite, inicia sesión en tu organización de Azure DevOps.

    ```
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO EShopOnWeb.Shared.1.0.0.nupkg
    ```
1. Espera a que se confirme la operación de inserción correcta del paquete.
1. Ve a la ventana del explorador web que muestra el portal de Azure DevOps y, en el panel de navegación vertical, selecciona **Artefactos**.
1. En el panel del centro de **Artefactos**, haz clic en la lista desplegable de la esquina superior izquierda y, en la lista de fuentes, selecciona la entrada **EShopOnWebShared**.

    > **Nota**: la fuente **EShopOnWebShared** debe incluir el paquete NuGet recién publicado.

1. Haz clic en el paquete NuGet para ver sus detalles.

#### Tarea 3: importar un paquete NuGet de código abierto a la fuente de paquetes de Azure DevOps

Además de desarrollar tus propios paquetes, ¿por qué no usar Nuget de código abierto (la biblioteca de paquetes DotNet https://www.nuget.org))? Con unos pocos millones de paquetes disponibles, siempre habrá algo útil para la aplicación.

En esta tarea, usaremos un paquete de ejemplo genérico "Hola mundo", pero puedes usar el mismo enfoque para otros paquetes de la biblioteca.

1. En la misma ventana de PowerShell, ejecuta el siguiente comando **nuget** para instalar el paquete de ejemplo:

```
.\nuget install HelloWorld -ExcludeVersion
```

1. Comprueba la salida del proceso de instalación. En la primera línea, figuran las diferentes fuentes que intentará descargar el paquete:
```
Feeds used:
  https://api.nuget.org/v3/index.json
  https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/EShopOnWebPFeed/nuget/v3/index.json
```
1. Después verás una salida adicional con respecto al proceso de instalación real.

```
Installing package 'Helloworld' to 'C:\eShopOnWeb\EShopOnWeb.Shared'.
  GET https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json
  OK https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json 114ms
MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
  GET https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json
  OK https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json 698ms

Attempting to gather dependency information for package 'Helloworld.1.3.0.17' with respect to project 'C:\eShopOnWeb\EShopOnWeb.Shared', targeting 'Any,Version=v0.0'
Gathering dependency information took 21 ms
Attempting to resolve dependencies for package 'Helloworld.1.3.0.17' with DependencyBehavior 'Lowest'
Resolving dependency information took 0 ms
Resolving actions to install package 'Helloworld.1.3.0.17'
Resolved actions to install package 'Helloworld.1.3.0.17'
Retrieving package 'HelloWorld 1.3.0.17' from 'nuget.org'.
  GET https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg
  OK https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg 133ms
Installed HelloWorld 1.3.0.17 from https://api.nuget.org/v3/index.json with content hash 1Pbk5sGihV5JCE5hPLC0DirUypeW8hwSzfhD0x0InqpLRSvTEas7sPCVSylJ/KBzoxbGt2Iapg72WPbEYxLX9g==.
Adding package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
Added package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
Successfully installed 'HelloWorld 1.3.0.17' to C:\eShopOnWeb\EShopOnWeb.Shared
Executing nuget actions took 686 ms
```
1. El paquete HelloWorld se ha instalado en una subcarpeta **HelloWorld**, en la carpeta EShopOnWeb.Shared. En el **Explorador de soluciones de Visual Studio**, ve al proyecto **EShopOnWeb.Shared** y busca la subcarpeta **HelloWorld**. Haz clic en la flecha pequeña situada a la izquierda de la subcarpeta para abrir la carpeta y la lista de archivos. 
1. Busca la subcarpeta **lib**, que tiene un archivo **signature.p7s** como prueba del origen del paquete. Después busca el archivo de paquete **HelloWorld.nupkg**. 

#### Tarea 4: cargar el paquete NuGet de código abierto en Azure Artifacts
Consideremos este paquete como "aprobado" para que nuestro equipo de DevOps lo reutilice, cargándolo en la fuente de paquetes de Azure Artifacts creada anteriormente.

1. En la nueva ventana de PowerShell, ejecuta los siguientes comandos para hacer lo siguiente:

 ```
.\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
```
    > **Nota**: aparece un mensaje de error.

```
Response status code does not indicate success: 409 (Conflict - 'HelloWorld 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'HelloWorld 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
```

Al crear la fuente de paquetes de artefactos de Azure DevOps, por diseño, se permiten **orígenes de nivel superior**, como nuget.org en el ejemplo de dotnet. Sin embargo, nada impide que el equipo de DevOps cree una fuente de paquetes **"solo interna"**.

1. Navega al portal de Azure DevOps y accede a **Artefactos**. Selecciona la fuente **EShopOnWebShared**. 
1. Haz clic en **Buscar orígenes de nivel superior**
1. En la ventana **Ir a un paquete de nivel superior**, selecciona **Nuget** como Tipo de paquete y escribe **HelloWorld** en el campo de búsqueda.
1. Para confirmar, presiona el botón **Buscar**.
1. Esto da como resultado una lista de todos los paquetes HelloWorld con las distintas versiones disponibles.
1. Haz clic en la **tecla de flecha izquierda** para volver a la fuente **EShopOnWebShared**.   
1. Haz clic en el engranaje para abrir **Configuración de la fuente**. En la página Configuración de la fuente, selecciona **Orígenes de nivel superior**. 
1. Observa los Administradores de paquetes de nivel superior para los distintos lenguajes de desarrollo. Selecciona **Nuget.org** de la lista. Presiona el botón **Eliminar** y, después, presiona el botón **Guardar**.

1. Con estos cambios guardados, será posible cargar el paquete **HelloWorld** mediante Nuget.exe desde la ventana de PowerShell; para ello, vuelve a introducir el siguiente comando:

```
 .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
```

> **Nota**: esto debería producir una carga correcta. 

```
Pushing HelloWorld.nupkg to 'https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/'...
  PUT https://pkgs.dev.azure.com/<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/
MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
  Accepted https://pkgs.dev.azure.com/pdtdemoworld<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/ 1645ms
Your package was pushed.
PS C:\eShopOnWeb\EShopOnWeb.Shared>
```

1. En el portal de Azure DevOps, **actualiza** la página Fuente de paquetes de artefactos. La lista de paquetes muestra el paquete personalizado **EShopOnWeb.Shared**, así como el paquete de origen público **HelloWorld**.
1. En la solución **EShopOnWeb.Shared** de Visual Studio, haz clic con el botón derecho en el proyecto **EShopOnWeb.Shared** y selecciona **Administrar paquetes de Nuget** del menú contextual. 
1. En la ventana Administrador de paquetes NuGet, confirma que el **origen del paquete** sea ** EShopOnWebShared**.
1. Haz clic en **Examinar** y espera a que se cargue la lista de paquetes Nuget.
1. Esta lista también mostrará el paquete personalizado **EShopOnWeb.Shared**, así como el paquete de origen público **HelloWorld**.

## Revisar

En este laboratorio, has aprendido a trabajar con Azure Artifacts siguiendo estos pasos:

- Has creado una fuente y te has conectado a ella.
- Has creado y publicado un paquete de NuGet.
- Has importado un paquete NuGet con desarrollo personalizado.
- Has importado un paquete de NuGet de origen público.
