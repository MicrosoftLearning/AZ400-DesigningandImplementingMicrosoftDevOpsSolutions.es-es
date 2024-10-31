---
lab:
  title: Administración de paquetes con Azure Artifacts
  module: 'Module 07: Design and implement a dependency management strategy'
---

# Administración de paquetes con Azure Artifacts

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOps:**: si aún no tienes una organización Azure DevOps que puedas usar para este laboratorio, crea una siguiendo las instrucciones disponibles en [Requisitos previos para el laboratorio AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configure el proyecto eShopOnWeb de ejemplo:** Si aún no tiene el proyecto eShopOnWeb de ejemplo que puede usar para este laboratorio, cree uno siguiendo las instrucciones disponibles en [Requisitos previos de laboratorio de AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- Visual Studio 2022 Community Edition está disponible en la [página de descargas de Visual Studio](https://visualstudio.microsoft.com/downloads/). La instalación de Visual Studio 2022 debe incluir **Desarrollo ASP.<nolink>NET y web, ** **desarrollo Azure**, y **desarrollo multiplataforma de .NET Core**.

- **SDK de .NET Core:** [Descargue e instale el SDK de .NET Core (2.1.400+)](https://go.microsoft.com/fwlink/?linkid=2103972)

- **Proveedor de credenciales de Azure Artifacts:** [Descargue e instale el proveedor de credenciales](https://go.microsoft.com/fwlink/?linkid=2099625).

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

En este ejercicio, configurarás los requisitos previos para el laboratorio.

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y deja los demás campos con los valores predeterminados. Haga clic en **Crear**.

    ![Captura de pantalla del panel Crear nuevo proyecto.](images/create-project.png)

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar un repositorio**. Seleccione **importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

    ![Captura de pantalla del panel Importar repositorio.](images/import-repo.png)

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

#### Tarea 4: Configuración de la solución eShopOnWeb en Visual Studio

En esta tarea, configurarás Visual Studio para prepararte para el laboratorio.

1. Asegúrese de que está viendo el proyecto de equipo de **eShopOnWeb** en el portal de Azure DevOps.

    > **Nota**: Para acceder directamente a la página del proyecto, vaya a la dirección URL [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb), donde el marcador de posición `<your-Azure-DevOps-account-name>` representa el nombre de la organización de Azure DevOps.

1. En el menú vertical del lado izquierdo del panel **eShopOnWeb**, haga clic en **Repositorios**.
1. En el panel **Archivos**, haz clic en **Clonar**, selecciona la flecha desplegable situada junto a **Clonar en VS Code** y, en el menú desplegable, selecciona **Visual Studio**.
1. Si el sistema pregunta si deseas continuar, haz clic en **Abrir**.
1. Inicia sesión con la cuenta de usuario que planeas usar en tu organización de Azure DevOps.
1. En la interfaz de Visual Studio, en la ventana emergente de **Azure DevOps**, acepte la ruta de acceso local predeterminada (C:\eShopOnWeb) y haga clic en **Clonar**. Esto importará el proyecto automáticamente en Visual Studio.
1. Deja abierta la ventana de Visual Studio para usarla en el laboratorio.

### Ejercicio 1: trabajo con Azure Artifacts

En este ejercicio, aprenderás a trabajar con Azure Artifacts siguiendo estos pasos:

- Cree una fuente y conéctese a ella.
- Cree y publique un paquete de NuGet.
- Importe un paquete de NuGet.
- Actualice un paquete de NuGet.

#### Tarea 1: creación y conexión a una fuente

En esta tarea, crearás una fuente y te conectarás a ella.

1. En la ventana del explorador web que muestra la configuración del proyecto en el portal de Azure DevOps, en el panel de navegación vertical, selecciona **Artefactos**.
1. Con el centro de **Artefactos** en pantalla, haz clic en **+ Crear fuente** en la parte superior del panel.

    > **Nota**: esta fuente será una colección de paquetes NuGet disponibles para los usuarios de la organización y se almacenará con con la fuente de NuGet pública como elemento del mismo nivel. El escenario de este laboratorio se centrará en el flujo de trabajo para usar Azure Artifacts, por lo que las decisiones de arquitectura y desarrollo reales son meramente ilustrativas.  Esta fuente incluirá funciones comunes que se pueden compartir entre proyectos de esta organización.

1. En el panel **Crear nueva fuente**, en el cuadro de texto **Nombre**, escribe **`eShopOnWebShared`**, en la sección **Ámbito**, selecciona la opción **Organización**, deja otras opciones con sus valores predeterminados y haz clic en **Crear**.

    > **Nota**: cualquier usuario que quiera conectarse a esta fuente de NuGet debe configurar su entorno.

1. Cuando vuelvas al centro de **Artefactos**, haz clic en **Conectar a la fuente**.
1. En el panel **Conectar para la fuente**, en la sección **NuGet**, selecciona **Visual Studio** y, en el panel de **Visual Studio**, copia la dirección URL de** origen**. `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`
1. Vuelve a la ventana de **Visual Studio**.
1. En la ventana de Visual Studio, haz clic en el menú **Herramientas**, en el menú desplegable, selecciona **Administrador de paquetes NuGet** y, en el menú en cascada, **Configuración del administrador de paquetes**.
1. En el cuadro de diálogo **Opciones**, haz clic en **Orígenes de paquete** de paquetes y haz clic en el signo de más para agregar un nuevo origen de paquete.
1. En la parte inferior del cuadro de diálogo, en el cuadro de texto **Nombre**, reemplace **Origen del paquete** por **eShopOnWebShared** y, en el cuadro de texto **Origen**, pegue la dirección URL que copió en el portal de Azure DevOps.
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
    | Nombre de proyecto | **eShopOnWeb.Shared** |
    | Location | acepte el valor predeterminado |
    | Solución | **Crear nueva solución** |
    | Nombre de la solución | **eShopOnWeb.Shared** |

    Active la casilla **Colocar solución y proyecto en el mismo directorio**.

1. Haga clic en Next. Acepte **.NET 8** como opción de marco.
1. Confirma la creación del proyecto presionando el botón **Crear**.
1. En la interfaz de Visual Studio, en el panel **Explorador de soluciones**, haz clic con el botón derecho en **Class1.cs**, en el menú contextual, selecciona **Eliminar** y, cuando se te pida confirmación, haz clic en **Aceptar**.
1. Pulsa **Ctrl+Shift+B** o **haz clic con el botón derecho en el proyecto** EShopOnWeb.Shared y selecciona **Compilar** para compilar el proyecto.
1. En tu estación de trabajo de laboratorio, abre el menú Inicio y busca **Windows PowerShell**. A continuación, en el menú en cascada, haz clic en **Abrir Windows PowerShell como administrador**.
1. En la ventana **Administrador: de Windows PowerShell**, vaya a la carpeta eShopOnWeb.Shared ejecutando el siguiente comando:

    ```text
    cd c:\eShopOnWeb\eShopOnWeb.Shared
    ```

    > **Nota**: La carpeta **eShopOnWeb.Shared** es la ubicación del archivo **eShopOnWeb.Shared.csproj**. Si eligió otra ubicación, vaya a esa ubicación en su lugar.

1. Ejecuta el comando siguiente para crear un archivo **.nupkg** a partir del proyecto.

    ```powershell
    dotnet pack .\eShopOnWeb.Shared.csproj
    ```

    > **Nota**: El comando **dotnet pack** compila el proyecto y crea un paquete NuGet en la carpeta **bin\Release**. Si no tienes una carpeta **Versión**, puedes usar la carpeta **Depurar** en su lugar.

    > **Nota**: Omite las advertencias que aparecen en la ventana **Administrador: Windows PowerShell**.

    > **Nota**: dotnet pack compila un paquete mínimo basado en la información que puede identificar del proyecto. Por ejemplo, tenga en cuenta que el nombre es **eShopOnWeb.Shared.1.0.0.nupkg**. Ese número de versión se ha recuperado del ensamblado.

1. En la ventana de PowerShell, ejecute el siguiente comando para abrir la carpeta **bin\Release**:

    ```powershell
    cd .\bin\Release
    ```

1. Ejecute lo siguiente para publicar el paquete en la fuente **eShopOnWebShared**:

    > **Importante**: Debe instalar el proveedor de credenciales para que el sistema operativo pueda autenticarse con Azure DevOps. Puede encontrar las instrucciones de instalación en [Proveedor de credenciales de Azure Artifacts](https://go.microsoft.com/fwlink/?linkid=2099625). Para instalar, ejecute el siguiente comando en la ventana de PowerShell: `iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) } -AddNetfx"`

    > **Nota**: Debes proporcionar una **clave de API**, que puede ser cualquier cadena no vacía. Usamos **az** aquí. Cuando se te solicite, inicia sesión en tu organización de Azure DevOps.

    ```powershell
    dotnet nuget push --source "eShopOnWebShared" --api-key az eShopOnWeb.Shared.1.0.0.nupkg
    ```

    > **Nota**: Si el símbolo del sistema no aparece, puede agregar el parámetro **--interactive** al comando.

1. Espera a que se confirme la operación de inserción correcta del paquete.
1. Ve a la ventana del explorador web que muestra el portal de Azure DevOps y, en el panel de navegación vertical, selecciona **Artefactos**.
1. En el panel de centro de **Artefactos**, haga clic en la lista desplegable de la esquina superior izquierda y, en la lista de fuentes, seleccione la entrada **eShopOnWebShared**.

    > **Nota**: La fuente **eShopOnWebShared** debe incluir el paquete NuGet recién publicado.

1. Haz clic en el paquete NuGet para ver sus detalles.

#### Tarea 3: importar un paquete NuGet de código abierto a la fuente de paquetes de Azure DevOps

Además de desarrollar sus propios paquetes, ¿por qué no usar la biblioteca de paquetes DotNet de NuGet (<https://www.nuget.org>) de código abierto? Con unos pocos millones de paquetes disponibles, siempre habrá algo útil para la aplicación.

En esta tarea, usaremos un paquete de ejemplo genérico "Newtonsoft.Json", pero puede usar el mismo enfoque para otros paquetes de la biblioteca.

1. En la misma ventana de PowerShell, vaya a la carpeta **eShopOnWeb.Shared** y ejecute el siguiente comando **dotnet** para instalar el paquete de ejemplo:

    ```powershell
    dotnet add package Newtonsoft.Json
    ```

1. Comprueba la salida del proceso de instalación. Muestra las diferentes fuentes que intentará descargar el paquete:

    ```powershell
    Feeds used:
      https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
      https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/eShopOnWebShared/nuget/v3/index.json
    ```

1. Después verás una salida adicional con respecto al proceso de instalación real.

    ```powershell
    Determining projects to restore...
    Writing C:\Users\AppData\Local\Temp\tmpxnq5ql.tmp
    info : X.509 certificate chain validation will use the default trust store selected by .NET for code signing.
    info : X.509 certificate chain validation will use the default trust store selected by .NET for timestamping.
    info : Adding PackageReference for package 'Newtonsoft.Json' into project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info :   GET https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
    info :   OK https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json 124ms
    info : Restoring packages for c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj...
    info :   GET https://api.nuget.org/v3/vulnerabilities/index.json
    info :   OK https://api.nuget.org/v3/vulnerabilities/index.json 84ms
    info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json
    info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json
    info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json 14ms
    info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json 30ms
    info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info : PackageReference for package 'Newtonsoft.Json' version '13.0.3' added to file 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
    info : Writing assets file to disk. Path: c:\eShopOnWeb\eShopOnWeb.Shared\obj\project.assets.json
    log  : Restored c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj (in 294 ms).
    ```

1. El paquete Newtonsoft.Json se instaló en los paquetes como **Newtonsoft.Json**. En el **Explorador de soluciones** de Visual Studio, vaya al proyecto **eShopOnWeb.Shared**, expanda Dependencias y observe **Newtonsoft.Json** en Paquetes. Haga clic en la flecha pequeña situada a la izquierda de Paquetes para abrir la carpeta y la lista de archivos.

#### Tarea 4: cargar el paquete NuGet de código abierto en Azure Artifacts

Consideremos este paquete como "aprobado" para que nuestro equipo de DevOps lo reutilice, cargándolo en la fuente de paquetes de Azure Artifacts creada anteriormente.

1. En Visual Studio, haga clic con el botón derecho en el nuevo paquete **Newtonsoft.Json** y seleccione **Abrir carpeta en el Explorador de archivos** en el menú contextual. Verá el nuevo paquete **Newtonsoft.Json** con la extensión **.nupkg**.
2. Copie la ruta de acceso completa desde la barra de direcciones de la ventana Explorador de archivos.
3. En la ventana de PowerShell, ejecute el siguiente comando reemplazando la ruta de acceso por la que copió:

    ```powershell
    dotnet nuget push --source "eShopOnWebShared" --api-key az C:\eShopOnWeb\eShopOnWeb.Shared\Newtonsoft.Json\newtonsoft.json.13.0.3.nupkg
    ```

    > **Nota**: aparece un mensaje de error.

    ```text
    Response status code does not indicate success: 409 (Conflict - 'Newtonsoft.Json 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'Newtonsoft.Json 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
    ```

Al crear la fuente de paquetes de artefactos de Azure DevOps, por diseño, se permiten **orígenes de nivel superior**, como nuget.org en el ejemplo de dotnet. Sin embargo, nada impide que el equipo de DevOps cree una fuente de paquetes **"solo interna"**.

1. Vaya al portal de Azure DevOps, vaya a **Artefactos** y seleccione la fuente **eShopOnWebShared**.
1. Haz clic en **Buscar orígenes de nivel superior**
1. En la ventana **Ir a un paquete de nivel superior**, selecciona **NuGet** como Tipo de paquete y escribe **`Newtonsoft.Json`** en el campo de búsqueda.
1. Para confirmar, presiona el botón **Buscar**.
1. Esto da como resultado una lista de todos los paquetes Newtonsoft.Json con las distintas versiones disponibles.
1. Haga clic en la **tecla de dirección izquierda** para volver a la fuente **eShopOnWebShared**.
1. Haz clic en el engranaje para abrir **Configuración de la fuente**. En la página Configuración de la fuente, selecciona **Orígenes de nivel superior**.
1. Observa los Administradores de paquetes de nivel superior para los distintos lenguajes de desarrollo. Selecciona **Galería de NuGet** en la lista. Presiona el botón **Eliminar** y, después, el botón **Guardar**.

1. Con estos cambios guardados, será posible cargar el paquete **Newtonsoft.Json** mediante el NuGet.exe desde la ventana de PowerShell, reiniciando el siguiente comando:

    ```text
     dotnet nuget push --source "eShopOnWebShared" --api-key az C:\eShopOnWeb\eShopOnWeb.Shared\Newtonsoft.Json\newtonsoft.json.13.0.3.nupkg
    ```

    > **Nota**: Esto debería dar lugar a una carga correcta.

    ```text
    Pushing newtonsoft.json.13.0.3.nupkg to 'https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/'...
    PUT https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/
    Accepted https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/_packaging/5faffb6c-018b-4452-a4d6-72c6bffe79db/nuget/v2/ 3160ms
    Your package was pushed.
    ```

1. En el portal de Azure DevOps, **actualiza** la página Fuente de paquetes de artefactos. La lista de paquetes muestra el paquete personalizado **eShopOnWeb.Shared**, así como el paquete de origen público **Newtonsoft.Json**.
1. En la solución **eShopOnWeb.Shared** de Visual Studio, haga clic con el botón derecho en el proyecto **eShopOnWeb.Shared** y seleccione **Administrar paquetes NuGet** en el menú contextual.
1. En la ventana Administrador de paquetes NuGet, valide que el **Origen del paquete** esté establecido en **eShopOnWebShared**.
1. Haz clic en **Examinar** y espera a que se cargue la lista de paquetes NuGet.
1. Esta lista también mostrará el paquete personalizado **eShopOnWeb.Shared**, así como el paquete de origen público **Newtonsoft.Json**.

## Revisar

En este laboratorio, has aprendido a trabajar con Azure Artifacts siguiendo estos pasos:

- Has creado una fuente y te has conectado a ella.
- Has creado y publicado un paquete de NuGet.
- Has importado un paquete NuGet con desarrollo personalizado.
- Has importado un paquete de NuGet de origen público.
