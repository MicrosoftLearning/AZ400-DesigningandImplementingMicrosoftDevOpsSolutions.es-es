---
lab:
  title: Configuración de grupos de agentes y descripción de estilos de canalización
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Configuración de grupos de agentes y descripción de estilos de canalización

# Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

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

En este ejercicio, configurarás el requisito previo del laboratorio, que consiste en el proyecto de equipo Parts Unlimited preconfigurado basado en una plantilla del Generador de demostraciones de Azure DevOps.

#### Tarea 1: configurar el proyecto del equipo

En esta tarea, usarás el Generador de demostraciones de Azure DevOps para generar un nuevo proyecto basado en la plantilla **PartsUnlimited**.

1. En el equipo de laboratorio, inicia un explorador web y ve al [Generador de demostraciones de Azure DevOps](https://azuredevopsdemogenerator.azurewebsites.net). Este sitio utilitario automatizará el proceso de creación de un nuevo proyecto de Azure DevOps dentro de la cuenta que se rellena previamente con el contenido (elementos de trabajo, repos, etc.) necesario para el laboratorio.

    > **Nota**: para obtener más información, consulta <https://docs.microsoft.com/en-us/azure/devops/demo-gen>.

1. Después, haz clic en **Iniciar sesión** y accede con la cuenta de Microsoft asociada a una suscripción de Azure DevOps.
1. Si es necesario, en la página **Generador de demostraciones de Azure DevOps**, haz clic en **Aceptar** para aceptar las solicitudes de permisos y acceder a la suscripción de Azure DevOps.
1. En la página **Crear proyecto**, en el cuadro de texto **Nombre del nuevo proyecto**, escribe **Configuración de grupos de agentes y descripción de estilos de canalización**, en la lista desplegable **Seleccionar organización**, selecciona tu organización de Azure DevOps y luego haz clic en **Elegir plantilla**.
1. En la página **Elegir una plantilla**, haz clic en la plantilla **PartsUnlimited** y luego en **Seleccionar plantilla**.
1. Haz clic en **Crear proyecto**.

    > **Nota**: espera a que finalice el proceso. Este proceso tardará alrededor de 2 minutos. En caso de que se produzca un error en el proceso, ve a la organización de DevOps, elimina el proyecto e inténtalo de nuevo.

1. En la página **Crear nuevo proyecto**, haz clic en **Navegar al proyecto**.

### Ejercicio 1: Creación de Azure Pipelines basadas en YAML

En este ejercicio, convertirás una canalización clásica de Azure en una basada en YAML.

#### Tarea 1: Creación de una canalización YAML de Azure DevOps

En esta tarea, crearás una canalización YAML de Azure DevOps basada en plantillas.

1. En el explorador web que muestra el portal de Azure DevOps con el proyecto **Configuración de grupos de agentes y descripción de estilos de canalización** abierto, en el panel de navegación vertical de la izquierda, haz clic en **Canalizaciones**.
1. En la pestaña **Reciente** del panel **Canalizaciones**, haz clic en **Nueva canalización**.
1. En el panel **¿Dónde está el código?**, selecciona **Git de Azure Repos**.
1. En el panel **Seleccionar un repositorio**, haz clic en **PartsUnlimited**.
1. En el panel **Revisar YAML de la canalización**, revisa la canalización de ejemplo, haz clic en el símbolo de intercalación hacia abajo situado junto al botón **Ejecutar** y haz clic en **Guardar**.

### Ejercicio 2: Administración de grupos de agentes de Azure DevOps

En este ejercicio, implementarás el agente de Azure DevOps autohospedado.

#### Tarea 1: Configuración de un agente de Azure DevOps autohospedado

En esta tarea, configurarás la máquina virtual de LOD como un agente de autohospedaje de Azure DevOps y la usarás para ejecutar una canalización de compilación.

1. Dentro de la máquina virtual de laboratorio (VM de laboratorio) o tu propio equipo, inicia un explorador web, navega hasta el [portal de Azure DevOps](https://dev.azure.com) e inicia sesión con la cuenta Microsoft asociada a tu organización de Azure DevOps.
1. En el portal de Azure DevOps, en la esquina superior derecha de la página de Azure DevOps, haz clic en el icono **Configuración de usuario**. En función de si tienes activadas o no las características de vista previa, deberías ver un elemento **Seguridad** o **Tokens de acceso personal** en el menú. Si ves **Seguridad**, haz clic en esta opción y luego, selecciona **Tokens de acceso personal**. En el panel **Tokens de acceso personal**, haz clic en **+ Nuevo token**.
1. En el panel **Crear un nuevo token de acceso personal**, haz clic en el vínculo **Mostrar todos los ámbitos**, especifica la siguiente configuración y haz clic en **Crear** (deja las demás opciones con sus valores predeterminados):

    | Configuración | Value |
    | --- | --- |
    | Nombre | **Configuración de grupos de agentes y descripción de estilos de canalización** |
    | Ámbito (definido personalizado) | **Grupos de agentes** (mostrar más opciones de ámbitos, aquí abajo, si es necesario)|
    | Permisos | **Leer y administrar** |

1. En el panel **Correcto**, copia el valor del token de acceso personal en el portapapeles.

    > **Nota**: Asegúrate de copiar el token. No podrás recuperarlo cuando cierres este panel.

1. En el panel **Correcto**, haz clic en **Cerrar**.
1. En el panel **Token de acceso personal** del portal de Azure DevOps, haz clic en el símbolo de **Azure DevOps** de la esquina superior izquierda y, luego, haz clic en la etiqueta **Configuración de la organización** de la esquina inferior izquierda.
1. En el lado izquierdo del panel **Información general**, en el menú vertical, en la sección **Canalizaciones**, haz clic en **Grupos de agentes**.
1. En el panel  **Grupos de agentes**, en la esquina superior derecha, haz clic en **Agregar grupo**.
1. En el panel **Agregar grupo de agentes**, en la lista desplegable **Tipo de grupo**, selecciona **Autohospedado**. En el cuadro de texto **Nombre**, escribe **az400m05l05a-pool** y, luego, haz clic en **Crear**.
1. De nuevo en el panel  **Grupos de agentes**, haz clic en la entrada que representa el grupo **az400m05l05a-pool** creado.
1. En la pestaña **Trabajos** del panel **az400m05l05a-pool**, haz clic en el botón **Nuevo agente**.
1. En el panel **Obtener el agente**, asegúrate de que están seleccionadas las pestañas **Windows** y **x64**, y haz clic en **Descargar** para descargar el archivo ZIP que contiene los archivos binarios del agente para descargarlos en la carpeta local **Descargas** de tu perfil de usuario.

    > **Nota**: Si recibes un mensaje de error en este punto que indica que la configuración actual del sistema impide descargar el archivo, en la ventana de Internet Explorer, en la esquina superior derecha, haz clic en el símbolo de engranaje que designa el encabezado de menú **Configuración**. En el menú desplegable, selecciona **Opciones de Internet**, en el cuadro de diálogo **Opciones de Internet**, haz clic en **Avanzadas**, en la pestaña **Avanzadas**, haz clic en **Restablecer**, en el cuadro de diálogo **Restablecer la configuración de Internet Explorer**, haz clic en **Restablecer** de nuevo, haz clic en **Cerrar** e inténtalo de nuevo.

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
    | Escribe la dirección URL del servidor | La dirección URL de la organización de Azure DevOps, en el formato ****<https://dev.azure.com/>`<organization_name>`, donde `<organization_name>` representa el nombre de la organización de Azure DevOps. |
    | Escribe el tipo de autenticación (presiona Entrar para PAT) | **Entrar** |
    | Introduce token de acceso personal | El token de acceso que has registrado anteriormente en esta tarea |
    | Introduce el grupo de agentes (presiona Entrar para el valor predeterminado) | **az400m05l05a-pool** |
    | Introduce el nombre del agente | **az400m05-vm0** |
    | Introduce la carpeta de trabajo (presiona Entrar para _work) | **Entrar** |
    | **(Solo si se muestra)** Introduce Realizar una tarea Unzip para las tareas de cada paso. (presiona ENTRAR para N) | **ADVERTENCIA**: solo presiona **Entrar** si aparece el mensaje|
    | Introduce ¿ejecutar agente como servicio? (Y/N) (presiona Entrar para N) | **Y** |
    | introduce habilitar SERVICE_SID_TYPE_UNRESTRICTED (Y/N) (presiona Entrar para N) | **Y** |
    | Introduce Cuenta de usuario que se usará para el servicio (presiona Entrar para NT AUTHORITY\NETWORK SERVICE) | **Entrar** |
    | Introduce si se va a impedir que el servicio se inicie inmediatamente después de que finalice la configuración (Y/N) (presiona Entrar para N) | **Entrar** |

    > **Nota**: puedes ejecutar el agente autohospedado como un servicio o un proceso interactivo. Es posible que quieras empezar con el modo interactivo, ya que esto simplifica la comprobación de la funcionalidad del agente. Para su uso en producción, debes considerar la posibilidad de ejecutar el agente como servicio o como un proceso interactivo con el inicio de sesión automático habilitado, ya que ambos conservan su estado en ejecución y asegúrate de que el agente se inicie automáticamente si se reinicia el sistema operativo.

1. Cambia a la ventana del explorador que muestra el portal de Azure DevOps y cierra el panel **Obtener el agente**.
1. Cuando vuelvas a la pestaña **Agentes** del panel **az400m05l05a-pool**, ten en cuenta que el agente recién configurado aparece con el estado **En línea**.
1. En la ventana del explorador web que muestra el portal de Azure DevOps, en la esquina superior izquierda, haz clic en la etiqueta **Azure DevOps**.
1. En la ventana del explorador que muestra la lista de proyectos, haz clic en el icono que representa el proyecto **Configuración de grupos de agentes y descripción de estilos de canalización**.
1. En el panel **Configurar grupos de agentes y descripción de los estilos de canalización**, en el panel de navegación vertical a la izquierda, en la sección **Canalizaciones**, haz clic en **Canalizaciones**.
1. En la pestaña **Recientes** del panel **Canalizaciones**, selecciona **PartsUnlimited** y, en el panel **PartsUnlimited**, selecciona **Editar**.
1. En el panel de edición **PartsUnlimited**, en la canalización basada en YAML existente, reemplaza la línea  `vmImage: windows-2019` que designa al grupo de agentes de destino el siguiente contenido, y designa el grupo de agentes autohospedado creado recientemente:

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```
    > **ADVERTENCIA**: ten cuidado con copiar y pegar, asegúrate de que tienes la misma sangría que se mostraba anteriormente.


1. Para `Task: NugetToolInstaller@0`, haz clic en **Configuración (vínculo que aparece arriba de la tarea en color gris),** modifica **Versión de NuGet.exe para instalar** > **4.0.0** y haz clic en **Agregar**.
1.  En el panel de edición **PartsUnlimited**, en la esquina superior derecha del panel, haz clic en **Guardar** y, en el panel **Guardar**, haz clic en **Guardar** nuevamente. Esto desencadenará automáticamente la compilación basada en esta canalización.
1.  En el portal de Azure DevOps, en el panel de navegación vertical del lado izquierdo, en la sección **Canalizaciones**, haz clic en **Canalizaciones**.
1.  En la pestaña **Recientes** del panel **Canalizaciones**, haz clic en la entrada **PartsUnlimited**; en la pestaña **Ejecuciones** del panel **PartsUnlimited**, selecciona la ejecución más reciente. En el panel **Resumen** de la ejecución, desplázate hacia abajo hasta la sección **Trabajos**, haz clic en **Fase 1** y controla el trabajo hasta que se complete correctamente.



## Revisar

En este laboratorio, aprenderás a implementar y usar agentes autohospedados con canalizaciones YAML.
