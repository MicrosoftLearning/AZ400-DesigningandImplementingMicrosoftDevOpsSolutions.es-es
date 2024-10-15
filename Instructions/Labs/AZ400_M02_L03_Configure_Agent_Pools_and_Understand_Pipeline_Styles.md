---
lab:
  title: Configurar grupos de agentes y descripción de los estilos de canalización
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Configuración de grupos de agentes y descripción de los estilos de canalización

## Manual de laboratorio para alumnos

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

## Tiempo estimado: 45 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y deja los demás campos con los valores predeterminados. Haga clic en **Crear**.

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar un repositorio**. Seleccione **Import** (Importar). En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

1. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - La carpeta **.github** contiene definiciones de flujo de trabajo de GitHub de YAML.
    - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

#### Tarea 3: (omitir si ya la has completado) Establecer la rama principal como rama predeterminada

1. Ve a **Repos>Ramas**.
1. Mantén el puntero sobre la rama **main** y haz clic en los puntos suspensivos a la derecha de la columna.
1. Haz clic en **Establecer como rama predeterminada**.

### Ejercicio 1: creación de Azure Pipelines basadas en YAML

En este ejercicio, crearás una canalización de compilación del ciclo de vida de la aplicación mediante una plantilla basada en YAML.

#### Tarea 1: Creación de una canalización YAML de Azure DevOps

En esta tarea, crearás una canalización YAML de Azure DevOps basada en plantillas.

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
   - 3 tareas dentro del trabajo de compilación:
   - Dotnet restore
   - Dotnet build
   - Dotnet publish

1. En el panel **Revisar YAML de la canalización**, haz clic en el símbolo de intercalación orientado hacia abajo situado junto al botón **Ejecutar** y haz clic en **Guardar**.

    > Nota: Estamos creando la definición de canalización por ahora, sin ejecutarla. Primero configurarás un grupo de agentes de Azure DevOps y ejecutarás la canalización en un ejercicio posterior. 

### Ejercicio 2: Administración de grupos de agentes de Azure DevOps

En este ejercicio, implementarás un agente de Azure DevOps autohospedado.

#### Tarea 1: Configuración de un agente de Azure DevOps autohospedado

En esta tarea, configurarás la máquina virtual de laboratorio como agente de autohospedaje de Azure DevOps y la usarás para ejecutar una canalización de compilación.

1. Dentro de la máquina virtual de laboratorio (VM de laboratorio) o tu propio equipo, inicia un explorador web, navega hasta el [portal de Azure DevOps](https://dev.azure.com) e inicia sesión con la cuenta Microsoft asociada a tu organización de Azure DevOps.

  > **Nota**: la máquina virtual del laboratorio debe tener instalado todo el software de requisitos previos necesario. Si lo va a instalar en su propio equipo, deberá instalar los SDK de .NET 8 o una versión superior necesarios para compilar el proyecto de demostración. Consulta [Descarga de .NET](https://dotnet.microsoft.com/download/dotnet).

1. En el portal de Azure DevOps, en la esquina superior derecha de la página de Azure DevOps, haz clic en el icono de **Configuración de usuario**. En función de si tienes activadas o no las características de vista previa, deberías ver un elemento **Seguridad** o **Tokens de acceso personal** en el menú. Si ves **Seguridad**, haz clic en esta opción y luego, selecciona **Tokens de acceso personal**. En el panel **Tokens de acceso personal**, haz clic en **+ Nuevo token**.
1. En el panel **Crear un nuevo token de acceso personal**, haz clic en el vínculo **Mostrar todos los ámbitos**, especifica la siguiente configuración y haz clic en **Crear** (deja las demás opciones con sus valores predeterminados):

    | Configuración | Value |
    | --- | --- |
    | Nombre | **eShopOnWeb** |
    | Ámbito (definido personalizado) | **Grupos de agentes** (mostrar más opciones de ámbitos, aquí abajo, si es necesario)|
    | Permisos | **Leer y administrar** |

1. En el panel **Correcto**, copia el valor del token de acceso personal en el portapapeles.

    > **Nota**: Asegúrate de copiar el token. No podrás recuperarlo cuando cierres este panel.

1. En el panel **Correcto**, haz clic en **Cerrar**.
1. En el panel **Token de acceso personal** del portal de Azure DevOps, haz clic en el símbolo de **Azure DevOps** en la esquina superior izquierda y, luego, haz clic en la etiqueta clic en **Configuración de la organización** en la esquina inferior izquierda.
1. En el lado izquierdo del panel **Información general**, en el menú vertical, en la sección **Canalizaciones**, haz clic en **Grupos de agentes**.
1. En el panel  **Grupos de agentes**, en la esquina superior derecha, haz clic en **Agregar grupo**.
1. En el panel **Agregar grupo de agentes**, en la lista desplegable **Tipo de grupo**, selecciona **Autohospedado**. En el cuadro de texto **Nombre**, escribe **az400m03l03a-pool** y después haz clic en **Crear**.
1. De vuelta en el panel **Grupos de agentes**, haz clic en la entrada que representa el grupo **az400m03l03a** recién creado.
1. En la pestaña **Trabajos** del panel **az400m03l03a-pool**, haz clic en el botón **Nuevo agente**.
1. En el panel **Obtener el agente**, asegúrate de que están seleccionadas las pestañas **Windows** y **x64**, y haz clic en **Descargar** para descargar el archivo ZIP que contiene los archivos binarios del agente para descargarlos en la carpeta **Descargas** locales de tu perfil de usuario.

    > **Nota**: Si recibes un mensaje de error en este punto que indica que la configuración actual del sistema te impide descargar el archivo, en la ventana Explorador, en la esquina superior derecha, haz clic en el símbolo de engranaje que designa el encabezado de menú **Configuración**, en el menú desplegable, selecciona **Opciones de Internet**, en el cuadro de diálogo **Opciones de Internet**, haz clic en **Avanzadas**, en la pestaña **Avanzadas**, haz clic en **Restablecer**, en el cuadro de diálogo **Restablecer configuración del explorador**, haz clic en **Restablecer** de nuevo, haz clic en **Cerrar** e intenta la descarga de nuevo.

1. Inicia Windows PowerShell como administrador y, en la consola **Administrador: Windows PowerShell** ejecuta las líneas siguientes para crear el directorio del **agente de C:\\** y extraer el contenido del archivo descargado en él.

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

1. En la misma consola **Administrador: Windows PowerShell**, ejecuta lo siguiente para configurar el agente:

    ```powershell
    .\config.cmd
    ```

1. Cuando te lo soliciten, especifica los valores de la siguiente configuración:

    | Configuración | Valor |
    | ------- | ----- |
    | Escribe la dirección URL del servidor | la dirección URL de la organización de Azure DevOps, en el formato **<https://dev.azure.com/>`<organization_name>`**, donde `<organization_name>` representa el nombre de la organización de Azure DevOps |
    | Escribe el tipo de autenticación (presiona Entrar para PAT) | **Entrar** |
    | Introduce token de acceso personal | El token de acceso que has registrado anteriormente en esta tarea |
    | Introduce el grupo de agentes (presiona Entrar para el valor predeterminado) | **az400m03l03a-pool** |
    | Introduce el nombre del agente | **az104-06-vm0** |
    | Introduce la carpeta de trabajo (presiona Entrar para _work) | **Entrar** |
    | **(Solo si se muestra)** Introduce Realizar una tarea Unzip para las tareas de cada paso. (presiona ENTRAR para N) | **ADVERTENCIA**: solo presiona **Entrar** si aparece el mensaje|
    | Introduce ¿ejecutar agente como servicio? (Y/N) (presiona Entrar para N) | **Y** |
    | introduce habilitar SERVICE_SID_TYPE_UNRESTRICTED (Y/N) (presiona Entrar para N) | **Y** |
    | Introduce Cuenta de usuario que se usará para el servicio (presiona Entrar para NT AUTHORITY\NETWORK SERVICE) | **Entrar** |
    | Introduce si se va a impedir que el servicio se inicie inmediatamente después de que finalice la configuración (Y/N) (presiona Entrar para N) | **Entrar** |

    > **Nota**: puedes ejecutar el agente autohospedado como un servicio o un proceso interactivo. Es posible que quieras empezar con el modo interactivo, ya que esto simplifica la comprobación de la funcionalidad del agente. Para su uso en producción, debes considerar la posibilidad de ejecutar el agente como servicio o como un proceso interactivo con el inicio de sesión automático habilitado, ya que ambos conservan su estado en ejecución y asegúrate de que el agente se inicie automáticamente si se reinicia el sistema operativo.

1. Cambia a la ventana del explorador que muestra el portal de Azure DevOps y cierra el panel **Obtener el agente**.
1. De nuevo en la pestaña **Agentes** del panel **az400m03l03a-pool**, observa que el agente recién configurado aparece con el estado **En línea**.
1. En la ventana del explorador web que muestra el portal de Azure DevOps, en la esquina superior izquierda, haz clic en la etiqueta **Azure DevOps**.
1. En la lista de proyectos, haga clic en el icono que representa el proyecto **eShopOnWeb**.
1. En el panel **eShopOnWeb**, en el panel de navegación vertical del lado izquierdo, en la sección **Canalizaciones**, haga clic en **Canalizaciones**.
1. En la pestaña **Recientes** del panel **Canalizaciones**, seleccione **eShopOnWeb** y, en el panel **eShopOnWeb**, seleccione **Editar**.
1. En el panel de edición de **eShopOnWeb**, en la canalización basada en YAML existente, reemplace la línea 13, que indica que `vmImage: ubuntu-latest` designa el grupo de agentes de destino al siguiente contenido, designando el grupo de agentes autohospedado recientemente creado:

    ```yaml
    name: az400m03l03a-pool
    demands:
    - Agent.Name -equals az400m03-vm0
    ```

    > **ADVERTENCIA**: Tenga cuidado con copiar y pegar, asegúrese de que tiene la misma sangría mostrada anteriormente.

    ![Sintaxis del grupo de YAML](images/m3/eshoponweb-ci-pr-pool_v1.png)

1. En la esquina superior derecha del panel de **eShopOnWeb**, haga clic en **Validar y guardar**. Esto desencadenará automáticamente la compilación basada en esta canalización.
1. En el portal de Azure DevOps, en el panel de navegación vertical del lado izquierdo, en la sección **Canalizaciones**, haz clic en **Canalizaciones**. En función de la configuración del laboratorio, es posible que la canalización le solicite permisos. Haga clic en **Permitir** para permitir que se ejecute la canalización. 
1. En la pestaña **Recientes** del panel **Canalizaciones**, haga clic en la entrada **eShopOnWeb**, en la pestaña **Ejecuciones** del panel **eShopOnWeb**, seleccione la ejecución más reciente, en el panel **Resumen** de la ejecución, desplácese hacia abajo hasta la parte inferior, en la sección **Trabajos**, haga clic en **Fase 1** y supervise el trabajo hasta su finalización correcta.

### Ejercicio 3: Eliminar los recursos usados en el laboratorio

1. Detén y quita el servicio del agente ejecutando `.\config.cmd remove` en el símbolo del sistema.
   - Se te pedirá que introduzcas de nuevo el token de acceso personal para quitar el agente de tu organización.
   - Si ya no tienes el token de acceso personal, puedes volver a generar el que creaste inicialmente en el ejercicio 2, tarea 1, paso 2.
1. Elimina el grupo de agentes.
1. Revoca el token PAT.
1. Revierta los cambios en el archivo **eshoponweb-ci-pr.yml**; para ello, vaya a él desde Repos/.ado/eshoponweb-ci-pr.yml, seleccione **Editar** y quite las líneas 13 a 15 (el fragmento de código del grupo de agentes) y cámbielo a `vmImage: ubuntu-latest`, tal como estaba inicialmente. (Esto se debe a que usará el mismo archivo de canalización de ejemplo más adelante en un ejercicio de laboratorio).

![Revertir el grupo de canalizaciones a la configuración de vmImage](images/m3/eshoponweb-ci-pr-vmimage_v1.png)

## Revisar

En este laboratorio, aprenderás a implementar y usar agentes autohospedados con canalizaciones YAML.
