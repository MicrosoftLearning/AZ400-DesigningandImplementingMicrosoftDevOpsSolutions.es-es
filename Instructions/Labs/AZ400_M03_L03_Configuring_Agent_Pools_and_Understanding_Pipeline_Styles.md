---
lab:
  title: Configuración de grupos de agentes y descripción de estilos de canalización
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Configuración de grupos de agentes y descripción de estilos de canalización

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

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

En este ejercicio, configurarás los requisitos previos para el laboratorio, que consta de un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ha terminado) crear y configurar el proyecto de equipo

En esta tarea, crearás un proyecto de Azure DevOps **eShopOnWeb** que usarán varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y deja los demás campos con los valores predeterminados. Haga clic en **Crear**.

#### Tarea 2: (omitir si ha terminado) importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que usarán varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar un repositorio**. Seleccione **Import** (Importar). En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL https://github.com/MicrosoftLearning/eShopOnWeb.git y haz clic en **Importar**:

2. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **.azure** contiene la infraestructura de Bicep y de ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - La carpeta **.github** contiene definiciones de flujo de trabajo de GitHub de YAML.
    - La carpeta **src** contiene el sitio web de .NET 7 que se usa en los escenarios de laboratorio.

### Ejercicio 1: creación de Azure Pipelines basadas en YAML

En este ejercicio, crearás una canalización de compilación del ciclo de vida de la aplicación mediante una plantilla basada en YAML.

#### Tarea 1: crear una canalización YAML de Azure DevOps

En esta tarea, crearás una canalización YAML de Azure DevOps basada en plantillas.

1. En el explorador web que muestra el portal de Azure DevOps con el proyecto **EShopOnWeb** abierto, en el panel de navegación vertical del lado izquierdo, haz clic en **Canalizaciones**.
2. En la pestaña **Reciente** del panel **Canalizaciones**, haz clic en **Nueva canalización**.
3. En la página **¿Dónde está el código?**, haz clic en **Git de Azure Repos**.
4. En el panel **Seleccionar un repositorio**, haz clic en **EShopOnWeb**.
5. En el panel **Configuración de la canalización**, haz clic en **Archivo YAML de Azure Pipelines existente**.
6. En **Seleccionar un archivo YAML existente**, selecciona **main** para la rama y **/.ado/eshoponweb-ci-pr.yml** para la ruta de acceso.
7. Haga clic en **Continuar**.
8. En el panel **Revisar YAML de la canalización**, revisa la canalización de ejemplo. Se trata de una canalización de compilación de aplicaciones .NET bastante sencilla, que hace lo siguiente:
- Una sola fase: compilación
- Un único trabajo: compilación
- 3 tareas dentro del trabajo de compilación:
- Dotnet Restore
- Dotnet Build
- Dotnet Publish

9. En el panel **Revisar YAML de la canalización**, haz clic en el símbolo de cursor de inserción orientado hacia abajo situado junto al botón **Ejecutar**, haz clic en **Guardar**.

    > Nota: estamos creando la definición de canalización por ahora, sin ejecutarla. Primero configurarás un grupo de agentes de Azure DevOps y ejecutarás la canalización en un ejercicio posterior. 

### Ejercicio 2: administración de grupos de agentes de Azure DevOps

En este ejercicio, implementarás un agente de Azure DevOps autohospedado.

#### Tarea 1: configurar un agente autohospedado de Azure DevOps

En esta tarea, configurarás la máquina virtual de laboratorio como agente autohospedado de Azure DevOps y la usarás para ejecutar una canalización de compilación.

1. Dentro de la máquina virtual de laboratorio (VM de laboratorio) o tu propio equipo, inicia un explorador web, ve al [Portal de Azure DevOps](https://dev.azure.com) e inicia sesión con la cuenta Microsoft asociada a tu organización de Azure DevOps.

  > **Nota**: la máquina virtual del laboratorio debe tener instalado todo el software de requisitos previos necesario. Si vas a realizar la instalación en tu propio equipo, deberás instalar los SDK de .NET 7.0.x o superior necesarios para compilar el proyecto de demo. Consulta [Descarga de .NET](https://dotnet.microsoft.com/download/dotnet).

1. En el portal de Azure DevOps, en la esquina superior derecha de la página de Azure DevOps, haz clic en el icono de **Configuración de usuario**, en función de si tienes activadas o no las características de vista previa, deberías ver un elemento **Seguridad** o **Tokens de acceso personal** en el menú, si ves **Seguridad**, haz clic en esta opción y luego selecciona **Tokens de acceso personal**. En el panel **Tokens de acceso personal**, haz clic en **+ Nuevo token**.
2. En el panel **Crear un token de acceso personal**, haz clic en el vínculo **Mostrar todos los ámbitos** y especifica la siguiente configuración y haz clic en **Crear** (deja las demás opciones con sus valores predeterminados):

    | Configuración | Value |
    | --- | --- |
    | Nombre | **eShopOnWeb** |
    | Ámbito (definido personalizado) | **Grupos de agentes** (mostrar más opciones de ámbitos, aquí abajo, si es necesario)|
    | Permisos | **Leer y administrar** |

3. En el panel **Correcto**, copia el valor del token de acceso personal en el Portapapeles.

    > **Nota**: asegúrate de copiar el token. No podrás recuperarlo una vez que cierres este panel.

4. En el panel **Correcto**, haz clic en **Cerrar**.
5. En el panel **Token de acceso personal** del portal de Azure DevOps, haz clic en el símbolo de **Azure DevOps** en la esquina superior izquierda y luego haz clic en la etiqueta **Configuración de la organización** en la esquina inferior izquierda.
6. En el lado izquierdo del panel **Información general**, en el menú vertical, en la sección **Canalizaciones**, haz clic en **Grupos de agentes**.
7. En el panel **Grupos de agentes**, en la esquina superior derecha, haz clic en **Agregar grupo**.
8. En el panel **Agregar grupo de agentes**, en la lista desplegable **Tipo de grupo**, selecciona **Autohospedado**, en el cuadro de texto **Nombre**, escribe **az400m03l03a-pool** y después haz clic en **Crear**.
9.  De vuelta en el panel **Grupos de agentes**, haz clic en la entrada que representa el grupo **az400m03l03a-pool** recién creado.
10. En la pestaña **Trabajos** del panel **az400m03l03a-pool**, haz clic en el botón **Nuevo agente**.
11. En el panel **Obtener el agente**, asegúrate de que están seleccionadas las pestañas **Windows** y **x64**, y haz clic en **Descargar** para descargar el archivo ZIP que contiene los archivos binarios del agente para descargarlos en la carpeta **Descargas** locales de tu perfil de usuario.

    > **Nota**: Si recibes un mensaje de error en este punto que indica que la configuración actual del sistema te impide descargar el archivo, en la ventana Explorador de la esquina superior derecha, haz clic en el símbolo de engranaje que designa el encabezado de menú **Configuración**. En el menú desplegable, selecciona **Opciones de Internet**, en el cuadro de diálogo **Opciones de Internet**, haz clic en **Avanzadas**, en la pestaña **Avanzadas**, haz clic en **Restablecer**, en el cuadro de diálogo **Restablecer configuración del explorador**, haz clic de nuevo en **Restablecer**, haz clic en **Cerrar** y vuelve a intentar la descarga.

12. Inicia Windows PowerShell como administrador y en la consola **Administrador: Windows PowerShell** ejecuta las líneas siguientes para crear el directorio **C:\\agent** y extraer el contenido del archivo descargado en él.

    ```powershell
    cd \
    mkdir agent ; cd agent
    $TARGET = Get-ChildItem "$Home\Downloads\vsts-agent-win-x64-*.zip"
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory($TARGET, "$PWD")
    ```

14. En la misma consola **Administrador: Windows PowerShell**, ejecuta lo siguiente para configurar el agente:

    ```powershell
    .\config.cmd
    ```

15. Cuando se te solicite, especifica los valores de la siguiente configuración:

    | Configuración | Valor |
    | ------- | ----- |
    | Especifica la dirección URL del servidor | la dirección URL de la organización de Azure DevOps, en el formato ****<https://dev.azure.com/>`<organization_name>`, donde `<organization_name>` representa el nombre de la organización de Azure DevOps. |
    | Escribe el tipo de autenticación (presiona Entrar para PAT) | **Entrar** |
    | Escribe un token de acceso personal | El token de acceso que has registrado anteriormente en esta tarea |
    | Especifica el grupo de agentes (presiona Entrar para el valor predeterminado) | **az400m03l03a-pool** |
    | Especifica el nombre del agente | **az400m03-vm0** |
    | Especifica la carpeta de trabajo (presiona Entrar para _work) | **Entrar** |
    | **(Solo si se muestra)** Especifica Realizar una tarea de descomprimir para las tareas de cada paso. (presiona ENTRAR para N) | **ADVERTENCIA**: solo presiona **Entrar** si se muestra el mensaje|
    | ¿Intro ejecuta agente como servicio? (Y/N) (presiona Entrar para N) | **Y** |
    | escribe enable SERVICE_SID_TYPE_UNRESTRICTED (Y/N) (presiona Entrar para N) | **Y** |
    | Especifica Cuenta de usuario que se utilizará para el servicio (presiona Entrar para NT AUTHORITY\NETWORK SERVICE) | **Entrar** |
    | Indique si desea evitar que el servicio se inicie inmediatamente después de finalizar la configuración. (Y/N) (presiona Entrar para N) | **Entrar** |

    > **Nota**: Puedes ejecutar el agente autohospedado como un servicio o un proceso interactivo. Es recomendable comenzar con el modo interactivo, ya que esto simplifica la comprobación de la funcionalidad del agente. Para su uso en producción, debes considerar la posibilidad de ejecutar el agente como servicio o como un proceso interactivo con el inicio de sesión automático habilitado, ya que ambos conservan su estado en ejecución y aseguran de que el agente se inicie automáticamente si se reinicia el sistema operativo.

16. Cambia a la ventana del explorador que muestra el portal de Azure DevOps y cierra el panel **Obtener el agente**.
17. De nuevo en la pestaña **Agentes** del panel **az400m03l03a-pool**, observa que el agente recién configurado aparece con el estado **En línea**.
18. En la ventana del explorador web que muestra el portal de Azure DevOps, en la esquina superior izquierda, haz clic en la etiqueta **Azure DevOps**.
19. En la ventana del explorador que muestra la lista de proyectos, haz clic en el icono que representa el proyecto **Configuración de grupos de agentes y descripción de estilos de canalización**.
20. En el panel **EShopOnWeb**, en el panel de navegación vertical del lado izquierdo, en la sección **Canalizaciones**, haz clic en **Canalizaciones**.
21. En la pestaña **Reciente** del panel **Canalizaciones**, selecciona **EShopOnWeb** y en el panel **EShopOnWeb**, selecciona **Editar**.
22. En el panel de edición **EShopOnWeb**, en la canalización basada en YAML existente, reemplaza la línea 13, que indica `vmImage: ubuntu-latest` que designa el grupo de agentes de destino, para designar el grupo de agentes autohospedado recién creado:

    ```yaml
    name: az400m03l03a-pool
    demands:
    - Agent.Name -equals az400m03-vm0
    ```

    > **ADVERTENCIA**: ten cuidado con copiar y pegar, asegúrate de usar la misma sangría que se muestra arriba.

23. En el panel de edición **EShopOnWeb**, en la esquina superior derecha del panel, haz clic en **Guardar** y en el panel **Guardar**, haz clic en **Guardar** de nuevo. Esto desencadenará automáticamente la compilación basada en esta canalización.
24. En el portal de Azure DevOps, en el panel de navegación vertical del lado izquierdo, en la sección **Canalizaciones**, haz clic en **Canalizaciones**.
25. En la pestaña **Reciente** del panel **Canalizaciones**, haz clic en la entrada **EShopOnWeb**. En la pestaña **Ejecuciones** del panel **EShopOnWeb**, selecciona la ejecución más reciente. En el panel **Resumen** de la ejecución, desplázate hacia abajo hasta la parte inferior, en la sección **Trabajos**, haz clic en **Fase 1** y supervisa el trabajo hasta que se complete correctamente.

### Ejercicio 3: Quitar los recursos usados en el laboratorio

1. Detén y quita el servicio del agente ejecutando `.\config.cmd remove` en el símbolo del sistema.
2. Elimina el grupo de agentes.
3. Revoca el token PAT.

## Revisar

En este laboratorio, aprendiste a implementar y usar agentes autohospedados con canalizaciones YAML.
