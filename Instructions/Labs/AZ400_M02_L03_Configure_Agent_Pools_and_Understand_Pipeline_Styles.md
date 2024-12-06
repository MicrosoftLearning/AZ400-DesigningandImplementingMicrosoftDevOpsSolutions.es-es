---
lab:
  title: Configurar grupos de agentes y descripción de los estilos de canalización
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Configuración de grupos de agentes y descripción de los estilos de canalización

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Página de descarga de [Git para Windows](https://gitforwindows.org/). Esto se instalará como parte de los requisitos previos para este laboratorio.

- [Visual Studio Code](https://code.visualstudio.com/). Esto se instalará como parte de los requisitos previos para este laboratorio.

## Introducción al laboratorio

Las canalizaciones basadas en YAML permiten implementar completamente CI/CD como código, en el que las definiciones de canalización residen en el mismo repositorio que el código que forma parte del proyecto de Azure DevOps. Las canalizaciones basadas en YAML admiten una amplia gama de características que forman parte de las canalizaciones clásicas, como solicitudes de incorporación de cambios, revisiones de código, historial, creación de ramas y plantillas.

Independientemente de la elección del estilo de canalización, para compilar el código o implementar la solución mediante Azure Pipelines, necesita un agente. Un agente hospeda recursos de proceso que ejecutan un trabajo a la vez. Los trabajos se pueden ejecutar directamente en la máquina host del agente o en un contenedor. Tiene una opción para ejecutar los trabajos mediante agentes hospedados por Microsoft, que se administran automáticamente, o bien de implementar un agente autohospedado que configure y administre personalmente.

En este laboratorio, aprenderá a implementar y usar agentes autohospedados con canalizaciones YAML.

## Objetivos

Después de completar este laboratorio, podrá:

- Implementar canalizaciones basadas en YAML.
- Implementar agentes autohospedados.

## Tiempo estimado: 30 minutos

## Instrucciones

### Ejercicio 0: (omitir si se ha realizado) Configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y deja los demás campos con los valores predeterminados. Haga clic en **Crear**.

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar un repositorio**. Seleccione **importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

1. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - La carpeta **.github** contiene definiciones de flujo de trabajo de GitHub de YAML.
    - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

#### Tarea 3: (omitir si ya la has completado) Establecer la rama principal como rama predeterminada

1. Ve a **Repos > Ramas**.
1. Mantén el puntero sobre la rama **main** y haz clic en los puntos suspensivos a la derecha de la columna.
1. Haz clic en **Establecer como rama predeterminada**.

### Ejercicio 1: Creación de agentes y configuración de grupos de agentes

En este ejercicio, creará una máquina virtual (VM) de Azure y la usará para crear un agente y configurar grupos de agentes.

#### Tarea 1: Creación de una VM de Azure y conexión a ella

1. En el explorador, abra Azure Portal desde `https://portal.azure.com`. Si se le solicita, inicie sesión con una cuenta con el rol Propietario en su suscripción de Azure.

1. En el cuadro **Buscar recursos, servicios y docuocumentos (G+/)**, escribe **`Virtual Machines`** y selecciónalo en la lista desplegable.

1. Seleccione el botón **Crear**.

1. Seleccione **Máquina virtual de Azure con configuración preestablecida**.

    ![Captura de pantalla de la creación de una máquina virtual con configuración preestablecida.](images/create-virtual-machine-preset.png)

1. Seleccione **Dev/Test** como entorno de carga de trabajo y **De uso general** como tipo de carga de trabajo.

1. Seleccione el botón **Seguir creando una VM**; en la pestaña **Aspectos básicos**, realice las siguientes acciones y, luego, seleccione **Administración**:

   | Configuración | Acción |
   | -- | -- |
   | Lista desplegable de **Suscripción** | Seleccione su suscripción a Azure. |
   | Sección **Grupo de recursos** | Cree un grupo de recursos denominado **rg-eshoponweb-agentpool**. |
   | Cuadro de texto **Nombre de máquina virtual**  | Introduce el nombre que prefieras, por ejemplo, **`eshoponweb-vm`**. |
   | Lista desplegable de **Región** | Puedes elegir tu región de [Azure](https://azure.microsoft.com/explore/global-infrastructure/geographies) más cercana. Por ejemplo, "eastus", "eastasia", "westus", etc. |
   | Lista desplegable **Opciones de disponibilidad** | Seleccione **No se requiere redundancia de la infraestructura**. |
   | Lista desplegable **Tipo de seguridad** | Seleccione la opción **Máquinas virtuales de inicio seguro**. |
   | Lista desplegable de **imágenes** | Seleccione la imagen **Windows Server 2022 Datacenter: Azure Edition - x64 Gen2**. |
   | Lista desplegable de **Tamaño** | Seleccione el tamaño **estándar** más barato para realizar pruebas. |
   | Cuadro de texto **Nombre de usuario** | Escriba el nombre de usuario que prefiera. |
   | Cuadro de texto **Contraseña** | Escriba la contraseña que prefiera. |
   | Sección **Puertos de entrada públicos** | Seleccione **Permitir los puertos seleccionados**. |
   | Lista desplegable **Seleccionar puertos de entrada** | Seleccione **RDP (3389)**. |

1. En la pestaña **Administración**, en la sección **Identidad**, seleccione la casilla de verificación **Habilitar identidad administrada asignada por el sistema** y, luego, seleccione **Revisar y crear**:

1. En la pestaña **Revisar y crear**, seleccione **Crear**.

   > **Nota**: Espere a que se complete el proceso de aprovisionamiento. Este proceso tardará alrededor de 2 minutos.

1. En Azure Portal, vaya a la página que muestra la configuración de la VM de Azure recién creada.

1. En la página de VM de Azure, selecciona **Conectar**, en el menú desplegable, selecciona **Conectar** y después selecciona **Descargar archivo RDP**.

1. Usa el archivo RDP descargado para establecer una sesión de Escritorio remoto en el sistema operativo que se ejecuta en la VM de Azure.

#### Tarea 2: Creación de un grupo de agentes

1. En la sesión de Escritorio remoto a la VM de Azure, inicie el explorador web de Microsoft Edge.

1. En el explorador web, vaya al portal de Azure DevOps en `https://aex.dev.azure.com` e inicie sesión para acceder a la organización.

   > **Nota**: Si es la primera vez que accedes al portal de Azure DevOps, es posible que tengas que crear tu perfil.

1. Abra el proyecto **eShopOnWeb** y seleccione **Configuración del proyecto** en el menú inferior izquierdo.

1. En **Canalizaciones > Grupos de agentes**, seleccione el botón **Agregar grupo**.

1. Elija el tipo de grupo **autohospedado**.

1. Proporcione un nombre para el grupo de agentes, como **eShopOnWebSelfPool** y agregue una descripción opcional.

1. Seleccione **Conceder permiso de acceso a todas las canalizaciones**.

   ![Captura de pantalla que muestra cómo agregar opciones de grupo de agentes con el tipo autohospedado.](images/create-new-agent-pool-self-hosted-agent.png)

   > **Nota:** No se recomienda conceder permiso de acceso a todas las canalizaciones para entornos de producción. Solo se usa en este laboratorio para simplificar la configuración de la canalización.

1. Seleccione el botón **Crear** para crear el grupo de agentes.

#### Tarea 3: Descarga y extracción de los archivos de instalación del agente

1. En el portal de Azure DevOps, seleccione el grupo de agentes recién creado y, luego, seleccione la pestaña **Agentes**.

1. Seleccione el botón **Nuevo agente** y, a continuación, el botón **Descargar** de la nueva ventana emergente **Descargar agente**.

   > **Nota**: Sigue las instrucciones para instalar el agente.

1. Inicie una sesión de PowerShell y ejecute los siguientes comandos para crear una carpeta denominada **agente**.

   ```powershell
   mkdir agent ; cd agent        
   ```

   > **Nota**: Asegúrate de que estás en la carpeta donde deseas instalar el agente, por ejemplo, C:\agent.

1. Ejecute el siguiente comando para extraer el contenido de los archivos del instalador del agente descargados:

   ```powershell
   Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-3.245.0.zip", "$PWD")
   ```

   > **Nota**: Si has descargado el agente en otra ubicación (o la versión descargada difiere), ajusta el comando anterior en consecuencia.

#### Tarea 4: Creación de un token PAT

> **Nota**: Antes de configurar el agente, debes crear un token PAT (a menos que tengas uno existente). Para crear un token PAT, siga estos pasos:

1. En la sesión de Escritorio remoto de la VM de Azure, abre otra ventana del explorador, ve al portal de Azure DevOps en `https://aex.dev.azure.com` y accede a tu organización y proyecto.

1. Seleccione **Configuración de usuario** en el menú superior derecho (directamente a la izquierda del icono de avatar del usuario).

1. Seleccione el elemento de menú **Token de acceso personal**.

   ![Captura de pantalla que muestra el menú Tokens de acceso personal.](images/personal-access-token-menu.png)

1. Seleccione el botón **Nuevo token**.

1. Proporcione un nombre para el token, como **eShopOnWebToken**.

1. Seleccione la organización de Azure DevOps para la que quiere usar el token.

1. Establezca la fecha de expiración del token (solo se usa para configurar el agente).

1. Seleccione el ámbito definido personalizado.

1. Seleccione mostrar todos los ámbitos.

1. Seleccione el ámbito **Grupos de agentes (Leer y administrar).**

1. Seleccione el botón **Crear** para crear el token.

1. Copie el valor del token y guárdelo en un lugar seguro (no podrá volver a verlo. Solo puede regenerar el token).

   ![Captura de pantalla que muestra la configuración del token de acceso personal.](images/personal-access-token-configuration.png)

   > [!IMPORTANT]
   > Use la opción de privilegios mínimos, **Grupos de agentes (leer y administrar)**, solo para la configuración del agente. Además, asegúrese de establecer la fecha de expiración mínima para el token si ese es el único propósito del token. Puede crear otro token con los mismos privilegios si necesita volver a configurar el agente.

#### Tarea 5: Configuración del agente

1. En la sesión de Escritorio remoto a la VM de Azure, vuelva a la ventana de PowerShell. Si es necesario, cambie del directorio actual a aquél en el que extrajo los archivos de instalación del agente anteriormente en este ejercicio.

1. Para configurar el agente a fin de que se ejecute desatendido, invoque el siguiente comando:

   ```powershell
   .\config.cmd
   ```

   > **Nota**: Si deseas ejecutar el agente de forma interactiva, usa `.\run.cmd` en su lugar.

1. Para configurar el agente, realice las siguientes acciones cuando se le solicite:

   - Escriba la dirección URL de la organización de Azure DevOps (**dirección URL del servidor**) en el formato `https://aex.dev.azure.com`{nombre de la organización}.
   - Acepta el tipo de autenticación predeterminado (**`PAT`**).
   - Escriba el valor del token PAT que creó en el paso anterior.
   - Escribe el nombre del grupo de agentes **`eShopOnWebSelfPool`** que has creado anteriormente en este ejercicio.
   - Escribe el nombre del agente **`eShopOnWebSelfAgent`**.
   - Acepte la carpeta de trabajo del agente predeterminada (_work).
   - Escriba **Y** para configurar el agente a fin de que se ejecute como servicio.
   - Escriba **Y** para habilitar SERVICE_SID_TYPE_UNRESTRICTED para el servicio del agente.
   - Escribe **`NT AUTHORITY\SYSTEM`** para establecer el contexto de seguridad para el servicio.

   > [!IMPORTANT]
   > En general, debe seguir el principio de privilegios mínimos al configurar el contexto de seguridad del servicio.

   - Acepte la opción predeterminada (**N**) para permitir que el servicio se inicie inmediatamente después de finalizar la configuración.

   ![Captura de pantalla que muestra la configuración del agente.](images/agent-configuration.png)

   > **Nota**: El proceso de configuración del agente tardará unos minutos en completarse. Una vez hecho esto, verás un mensaje que indica que el agente se está ejecutando como servicio.

   > [!IMPORTANT] Si ves un mensaje de error que indica que el agente no se está ejecutando, es posible que tengas que iniciar el servicio manualmente. Para ello, abre el applet **Servicios** del panel de control de Windows, busca el servicio denominado **Agente de Azure DevOps (eShopOnWebSelfAgent)** e inícialo.

   > [!IMPORTANT] Si el agente no se inicia, es posible que tengas que elegir una carpeta diferente para el directorio de trabajo del agente. Para ello, vuelve a ejecutar el script de configuración del agente y elige otra carpeta.

1. Para comprobar el estado del agente, cambie al explorador web que muestra el portal de Azure DevOps, vaya al grupo de agentes y haga clic en la pestaña **Agentes**. Debería ver el nuevo agente en la lista.

   ![Captura de pantalla que muestra el estado del agente.](images/agent-status.png)

   > **Nota**: Para obtener más información sobre los agentes de Windows, consulta: [Agentes de Windows autohospedados](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent)

   > [!IMPORTANT]
   > Para que el agente pueda compilar e implementar recursos de Azure desde las canalizaciones de Azure DevOps (que configurará en los próximos laboratorios), debe instalar la CLI de Azure en el sistema operativo de la máquina virtual de Azure que hospeda el agente.

1. Inicie un explorador web y vaya a la página [Instalación de la CLI de Azure en Windows](https://learn.microsoft.com/cli/azure/install-azure-cli-windows?tabs=azure-cli#install-or-update).

1. Descargue e instale la CLI de Azure.

1. (Opcional) Si lo prefieres, ejecuta el siguiente comando de PowerShell para instalar la CLI de Azure:

   ```powershell
   $ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; Remove-Item .\AzureCLI.msi
   ```

   > **Nota**: Si usas una versión diferente de la CLI de Azure, es posible que tengas que ajustar el comando anterior en consecuencia.

1. En el explorador web, ve a la página del instalador del SDK de Microsoft .NET 8.0 en `https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-8.0.403-windows-x64-installer`.

   > [!IMPORTANT]
   > Debes instalar el SDK de .NET 8.0 (o superior) en la VM de Azure que hospeda el agente. Esto es necesario para compilar la aplicación eShopOnWeb en los próximos laboratorios. Cualquier otra herramienta o SDK necesarios para la compilación de la aplicación también debe instalarse en la VM de Azure.

1. Descarga e instala el SDK de Microsoft .NET 8.0.

### Ejercicio 2: Creación de Azure Pipelines basadas en YAML

En este ejercicio, crearás una canalización de compilación del ciclo de vida de la aplicación mediante una plantilla basada en YAML.

#### Tarea 1: Creación de una canalización YAML de Azure DevOps

En esta tarea, crearás una canalización basada en YAML para el proyecto **eShopOnWeb**.

1. En el explorador web en el que se muestra el portal de Azure DevOps con el proyecto **eShopOnWeb** abierto, en el panel de navegación vertical del lado izquierdo, haga clic en **Canalizaciones**.
1. Haga clic en el botón **Crear canalización**, si aún no ha creado ninguna canalización, o haga clic en **Nueva canalización** para crear una nueva.

1. En el panel **¿Dónde está el código?**, selecciona **Git de Azure Repos**.
1. En el panel **Seleccionar un repositorio**, haz clic en **eShopOnWeb**.
1. En el panel **Configure su canalización**, haz clic en **Archivo YAML de Azure Pipelines existente**.
1. En **Seleccionar un archivo YAML existente**, selecciona **principal** para la rama y **/.ado/eshoponweb-ci-pr.yml** para la ruta de acceso.
1. Haga clic en **Continue**.
1. En el panel **Revisar YAML de la canalización**, revisa la canalización de ejemplo. Se trata de una canalización de compilación de aplicaciones .NET bastante sencilla, que hace lo siguiente:

   - Una sola fase: Compilar
   - Un único trabajo: Compilar
   - 4 tareas dentro del trabajo de compilación:
   - Dotnet restore
   - Dotnet build
   - Dotnet Test
   - Dotnet publish

1. En el panel **Revisar YAML de la canalización**, haz clic en el símbolo de intercalación orientado hacia abajo situado junto al botón **Ejecutar** y haz clic en **Guardar**.

    > **Nota**: Estamos creando la definición de canalización por ahora, sin ejecutarla. Primero configurarás un grupo de agentes de Azure DevOps y ejecutarás la canalización en un ejercicio posterior.

#### Tarea 2: Actualizar la canalización de YAML con el grupo de agentes autohospedado

1. En el portal de Azure DevOps, ve al proyecto **eShopOnWeb** y selecciona **Canalizaciones** en el menú de la izquierda.
1. Haz clic en el botón **Editar** de la canalización que has creado en la tarea anterior.
1. En el panel de edición de **eShopOnWeb**, en la canalización basada en YAML existente, elimina la línea 13, que indica que **vmImage: ubuntu-latest** designa el grupo de agentes de destino al siguiente contenido, designando el grupo de agentes autohospedado creado recientemente:

    ```yaml
    pool: 
      name: eShopOnWebSelfPool
      demands: Agent.Name -equals eShopOnWebSelfAgent
    ```

    > **ADVERTENCIA**: Tenga cuidado con copiar y pegar, asegúrese de que tiene la misma sangría mostrada anteriormente.

    ![Captura de pantalla que muestra la sintaxis del grupo de YAML.](images/eshoponweb-ci-pr-agent-pool.png)

1. En la esquina superior derecha del panel de **eShopOnWeb**, haga clic en **Validar y guardar**. A continuación, haga clic en **Save**(Guardar).
1. En la esquina superior derecha del panel de **eShopOnWeb**, haz clic en **Ejecutar canalización**.

    > **Nota**: La canalización se ejecutará en el grupo de agentes autohospedado que has creado en el ejercicio anterior.
1. Abre la ejecución de la canalización y supervisa el trabajo hasta que se complete correctamente.

    > **Nota**: Si recibes un mensaje de permisos, haz clic en **Permitir** para permitir que se ejecute la canalización.

1. Una vez completada la ejecución de la canalización, revisa la salida y comprueba que la canalización se ha ejecutado correctamente.

## Revisar

En este laboratorio, aprenderás a implementar y usar agentes autohospedados con canalizaciones YAML.
