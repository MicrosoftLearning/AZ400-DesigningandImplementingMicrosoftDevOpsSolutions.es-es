---
lab:
  title: Configuración y ejecución de pruebas funcionales
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Configuración y ejecución de pruebas funcionales

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility?view=azure-devops).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique una suscripción de Azure existente o cree una.

- Compruebe que tenga una cuenta de Microsoft o Azure AD con el rol Propietario en la suscripción de Azure y el rol Administrador global en el inquilino de Azure AD asociado a la suscripción de Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Introducción al laboratorio

[Selenium](http://www.seleniumhq.org/): se trata de un marco portable de pruebas de software de código abierto para aplicaciones web. Puede funcionar en casi todos los sistemas operativos. Admite todos los exploradores modernos y varios lenguajes, incluidos .NET (C#) y Java.

Este laboratorio le enseñará a ejecutar casos de prueba de Selenium en una aplicación web de C# como parte de la canalización de versión de Azure DevOps.

## Objetivos

Después de completar este laboratorio, podrá:

- Configure un agente de Azure DevOps autohospedado.
- Configure la canalización de versión.
- Desencadene la compilación y la versión.
- Ejecute pruebas en Chrome y Firefox.

## Tiempo estimado: 60 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, que incluyen el proyecto de equipo DevOps Unlimited preconfigurado basado en una plantilla de generador de demostración de Azure DevOps y recursos de Azure.

#### Tarea 1: configurar el proyecto del equipo

En esta tarea, usarás el generador de demostraciones de Azure DevOps para generar un nuevo proyecto basado en la **plantilla Selenium**.

1. En el equipo de laboratorio, inicia un explorador web y ve al [Generador de demostraciones de Azure DevOps](https://azuredevopsdemogenerator.azurewebsites.net). Este sitio de utilidad automatizará el proceso de creación de un nuevo proyecto de Azure DevOps dentro de la cuenta que se rellena previamente con contenido (elementos de trabajo, repositorios, etc.) necesarios para el laboratorio.

    > **Nota**: Para obtener más información sobre el sitio, consulta [¿Qué es el generador de demostraciones de Azure DevOps Services?](https://docs.microsoft.com/azure/devops/demo-gen).

2. Después, haz clic en **Iniciar sesión** y accede con la cuenta de Microsoft asociada a una suscripción de Azure DevOps.
3. Si es necesario, en la página **Generador de demostraciones de Azure DevOps**, haz clic en **Aceptar** para aceptar las solicitudes de permisos y acceder a la suscripción de Azure DevOps.
4. En la página **Crear nuevo proyecto**, en el cuadro de texto **Nuevo nombre de proyecto**, escribe **Configurar y ejecutar pruebas** funcionales, en la **lista desplegable Seleccionar organización**, selecciona la organización de Azure DevOps y después haz clic en **Elegir plantilla**.
5. En la lista de plantillas, en la barra de herramientas, haz clic en **DevOps Labs**, selecciona la **plantilla Selenium** y haz clic en **Seleccionar plantilla**.
6. Cuando vuelvas la página **Crear nuevo proyecto**, haz clic en **Crear proyecto**.

    > **Nota**: espera a que finalice el proceso. Este proceso tardará alrededor de 2 minutos. En caso de que se produzca un error en el proceso, ve a la organización de DevOps, elimina el proyecto e inténtalo de nuevo.

7. En la página **Crear nuevo proyecto**, haz clic en **Navegar al proyecto**.

#### Tarea 2: Crear recursos de Azure

En esta tarea, aprovisionarás una máquina virtual de Azure que ejecute Windows Server 2016 junto con SQL Express 2017, Chrome y Firefox.

1. Haz clic aquí, en el enlace de **[Implementación en Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json)**. Te redirigirá automáticamente al explorador a la hoja **Implementación personalizada** en Azure Portal.
2. Si se te pide, inicia sesión con una cuenta de usuario que tenga el rol de propietario en la suscripción a Azure y el rol de administrador global en el inquilino de Azure AD asociado a esa suscripción.
3. En la hoja **Implementación personalizada**, selecciona **Editar plantilla**.
4. En la hoja **Editar plantilla**, busca la línea `"https://raw.githubusercontent.com/microsoft/azuredevopslabs/master/labs/vstsextend/selenium/armtemplate/chrome_firefox_VSTSagent_IIS.ps1"`, reemplázala por `"https://raw.githubusercontent.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/master/Allfiles/Labs/11b/chrome_firefox_VSTSagent_IIS.ps1"`, y haz clic en **Guardar**.
5. De nuevo en la hoja **Implementación personalizada**, configure las opciones siguientes:

    | Configuración | Value |
    | --- | --- |
    | Subscription | nombre de la suscripción de Azure que usa en este laboratorio |
    | Resource group | el nombre de un nuevo grupo de recursos **az400m11l02-RG** |
    | Region | el nombre de la región de Azure en la que implementaste recursos de este laboratorio |
    | Nombre de máquina virtual | **az40011bvm** |

6. Haga clic en **Revisar y crear** y, a continuación, en **Crear**.

    > **Nota**: espera a que finalice el proceso. Este proceso tardará aproximadamente 15 minutos.

### Ejercicio 1: Implementación de pruebas de Selenium mediante un agente de Azure DevOps autohospedado

En este ejercicio, implementarás pruebas de Selenium mediante un agente de Azure DevOps autohospedado.

#### Tarea 1: Configura un agente de Azure DevOps autohospedado

En esta tarea, configurarás un agente autohospedado mediante la máquina virtual que implementaste en el ejercicio anterior. Selenium requiere que el agente se ejecute en el modo interactivo para ejecutar las pruebas de IU.

1. En el explorador web donde se muestra Azure Portal, busca y selecciona **Máquinas virtuales** y, en la hoja **Máquinas virtuales**, selecciona **az40011bvm**.
2. En la hoja **az40011bvm**, selecciona **Conectar**, en el menú desplegable, selecciona **RDP**, en la **pestaña RDP** de la **hoja az40011bvm \| Conectar**, selecciona **Descargar archivo** RDP y abre el archivo descargado.
3. Cuando se le solicite, inicie sesión con las siguientes credenciales:

    | Configuración | Valor |
    | --- | --- |
    | Nombre de usuario | **vmadmin** |
    | Contraseña | **P2ssw0rd@123** |

4. Dentro de la sesión de Escritorio remoto a **az40011bvm**, abre una ventana del explorador web Chrome, ve a **<https://dev.azure.com>** la organización de Azure DevOps e inicia sesión.
5. En la esquina inferior izquierda del portal de **Azure DevOps**, haz clic en **Configuración de la organización**.
6. En el menú vertical del lado izquierdo de la página, en la sección **Canalizaciones**, haz clic en **Grupos de agentes**.
7. En el panel **Grupos de agentes**, haz clic en **Predeterminado**.
8. En el **panel Predeterminado**, haz clic en **Nuevo agente**.
9. En el panel **Obtener el agente**, asegúrate de que la pestaña **Windows** y la sección **x64** están seleccionadas y después haz clic en **Descargar**.
10. Inicia el Explorador de archivos, crea un directorio **C:\\AzAgent** y extrae contenido del archivo ZIP del agente descargado que reside en la carpeta **Descargas** en este directorio.
11. Dentro de la sesión de Escritorio remoto a **az40011bvm**, haz clic con el botón derecho en el menú **Inicio** y haz clic en **Símbolo del sistema (Administración)**.
12. En la ventana ** Administración: símbolo del sistema**, ejecuta lo siguiente para iniciar la instalación de los archivos binarios del agente:

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```

13. En la ventana **Administración: símbolo del sistema**, cuando se te pida que **escribas la dirección URL** del servidor, escribe**https://dev.azure.com/\<your-DevOps-organization-name\>**, donde\<your-DevOps-organization-name\>**** representa el nombre de la organización de Azure DevOps y pulsa la tecla ** Enter.**
14. En la ventana ** Administración: símbolo del sistema**, cuando se **te solicite Introducir tipo de autenticación (pulse Enter para PAT),** pulsa la **tecla** Enter.
15. En la ventana ** Administración: símbolo del sistema**, cuando se te solicite ** Introduce el token de acceso personal**, cambia al portal de Azure DevOps, cierra el panel **Obtener el agente**, en la esquina superior derecha de la página de Azure DevOps, haz clic en el icono **Configuración de usuario**, en el menú desplegable, haz clic en **Tokens de acceso personal**, en el **panel Tokens de acceso personal** y haz clic en **+ Nuevo token**.
16. En la hoja **Crear un nuevo acceso token personal**, especifica las siguientes opciones de configuración (deja las demás con los valores predeterminados) y haz clic en **Crear**:

    | Configuración | Value |
    | --- | --- |
    | Nombre | **Configuración y ejecución de pruebas funcionales** |
    | Ámbitos | **Definido por el usuario** |
    | Ámbitos | Haz clic en **Mostrar opciones** (en la parte inferior de la ventana) |
    | Ámbitos | **Grupos de agentes** - **Leídos y administrados** |

17. En el panel **Correcto**, copia el valor del token de acceso personal en el portapapeles.

    > **Nota**: Asegúrate de copiar el token. No podrás recuperarlo cuando cierres este panel.

18. En el panel **Correcto**, haz clic en **Cerrar**.
19. Vuelve a la ventana **Administración: símbolo del sistema** y pega el contenido del Portapapeles y pulsa la **tecla Enter**.
20. En la **ventana Administración: símbolo del sistema**, cuando se **te solicite Introducir grupo de agentes (pulsa enter para el valor predeterminado),** pulsa la **tecla** Enter.
21. En la **ventana Administración: símbolo del sistema**, cuando se **te solicite Introducir el nombre del agente (pulsa enter para az40011bvm),** pulsa la **tecla Enter**.
22. En la ventana ** Administración: símbolo del sistema**, cuando se **te solicite Introducir la carpeta de trabajo (pulsa enter para _work),** pulsa la **tecla** Enter.
23. En la ventana **Administración: símbolo del sistema**, cuando se **te solicite Introducir en ejecución del agente como servicio (S/N) (pulsa enter para N),** pulsa la **tecla** Enter.
24. En la ventana **Administrador: Símbolo del sistema**, cuando se te solicite **entrar, configurar el inicio automático y ejecutar el agente en el inicio (S/N) (presiona Entrar para N,** pulsa la **tecla Entrar**.
25. Una vez registrado el agente, en la ventana **Administrador: Símbolo del sistema**, escribe **run.cmd** y presiona **Entrar** para iniciar el agente.

    > **Nota**: También debes instalar Dac Framework que usa la aplicación que vas a implementar más adelante en el laboratorio.

26. En la sesión de Escritorio remoto en az40011bvm****, inicia otra instancia del explorador web, navega hasta la página de descarga de [Microsoft SQL Server Data-Tier Application Framework (18.2)](https://www.microsoft.com/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions) y haz clic en **Descargar**.
27. En **Elige la descarga que desee**, selecciona la casilla **N\x64\DacFramework.msi** y haz clic en **Siguiente**. Esto desencadenará la descarga automática del archivo **DacFramework.msi**.
28. Una vez completada la descarga del archivo **DacFramework.msi**, úsalo para ejecutar la instalación de Microsoft SQL Server Data-Tier Application Framework con la configuración predeterminada.

#### Tarea 2: Configuración de una canalización de la versión

En esta tarea, configurarás la canalización de la versión.

> **Nota**: La máquina virtual de Azure tiene el agente configurado para implementar las aplicaciones y ejecutar casos de prueba de Selenium. La definición de versión usa **[Fases](https://docs.microsoft.com/vsts/build-release/concepts/process/phases)** para implementar en servidores de destino.

1. Dentro de la sesión de Escritorio remoto en az40011bvm****, en la ventana del explorador que muestra el **portal de Azure DevOps**, haz clic en el **símbolo de Azure DevOps** en la esquina superior izquierda.
2. En el panel que muestra los proyectos de la organización, haz clic en el icono que representa el proyecto **Configuración y ejecución de pruebas funcionales**.
3. En el  panel **Configuración y ejecución de pruebas funcionales**, en el panel de navegación vertical, selecciona **Canalizaciones**, en la **sección Canalizaciones**, haz clic en **Versiones** y, luego, en el panel **Selenium**, haz clic en **Editar**.
4. En el panel **Todas las canalizaciones > Selenium**, haz clic en el encabezado de la pestaña **Tareas** y, en el menú desplegable, haz clic en **Desarrollo**.
5. En la lista de tareas de la fase **Desarrollo**, revisa las fases de implementación **implementación de IIS**, **implementación de SQL** y **ejecución de pruebas de Selenium**.

   - **Fase de implementación de IIS**: en esta fase, implementamos la aplicación en la máquina virtual mediante las siguientes tareas:

     - **Administración de aplicaciones web de IIS**: esta tarea se ejecuta en la máquina de destino donde hemos registrado el agente. Crea un *sitio web* y un *grupo de aplicaciones* con el nombre **PartsUnlimited** que se ejecutan en el puerto **82**, [**http://localhost:82**](http://localhost:82)
     - **Implementación de aplicaciones web de IIS**: esta tarea implementa la aplicación en el servidor IIS mediante **Web Deploy**.

   - **Fase de implementación de base de datos**: en esta fase, usamos la tarea [**Implementación de la base de datos de SQL Server**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md) para implementar[** el archivo dacpac**](https://docs.microsoft.com/sql/relational-databases/data-tier-applications/data-tier-applications) en el servidor de base de datos.

   - **Ejecución de pruebas de Selenium**: la ejecución de ** de pruebas de IU** como parte del proceso de lanzamiento nos permite detectar cambios inesperados. La configuración de pruebas automatizadas basadas en el explorador mejora la calidad de la aplicación, sin tener que hacerlo manualmente. En esta fase, ejecutaremos pruebas de Selenium en la aplicación web implementada. En las tareas posteriores se describe el uso de Selenium para probar los sitios web en la canalización de versión.

     - **Instalador de la plataforma de pruebas de Visual Studio**: la tarea del [instalador de la plataforma de pruebas de Visual Studio](https://docs.microsoft.com/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts) adquirirá la plataforma de prueba de Microsoft de nuget.org o de una fuente especificada y la añadirá a la memoria caché de herramientas. Satisface los requisitos de** vstest** para que cualquier tarea de prueba de Visual Studio posterior en una compilación o canalización de versión se pueda ejecutar sin necesidad de una instalación completa de Visual Studio en la máquina de agente.
     - **Ejecutar pruebas de interfaz de usuario de Selenium**: esta [tarea ](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md) usa **vstest.console.exe** para ejecutar los casos de prueba de Selenium en las máquinas del agente.

6. En el panel **Todas las canalizaciones > Selenium**, haz clic en la fase **Implementación de IIS**  y, en el panel **Trabajo de agente**, comprueba que está seleccionado el grupo de agentes **predeterminado**.
7. Repite el paso anterior para las fases **Implementación de SQL** y **Ejecución de fases de Selenium**. Haz clic en el botón **Guardar** para guardar los cambios.

#### Tarea 3: Desencadenar la compilación y la versión

En esta tarea, desencadenaremos la **compilación** para compilar scripts de Selenium C# junto con la aplicación web. Los archivos binarios resultantes se copian en el agente autohospedado y los scripts de Selenium se ejecutan como parte de la **versión** automatizada.

1. Dentro de la sesión de Escritorio remoto en az40011bvm****, en la ventana del explorador que muestra el portal de **Azure DevOps**, en el panel de navegación vertical, en la sección **Canalizaciones**, haz clic en **Canalizaciones** y, luego, en el panel **Canalizaciones**, haz clic en **Selenium**.
2. En el panel **Selenium**, haz clic en **Ejecutar canalización** y, en **Ejecutar canalización**, haz clic en **Ejecutar**.

    > **Nota**: Esta compilación publicará los artefactos de prueba en Azure DevOps, que se usarán en la versión.

    > **Nota**: Cuando la compilación se haya realizado correctamente, se generará una nueva versión.

3. En el panel de ejecuciones de canalización, en la sección **Trabajos**, haz clic en **Fase 1** y supervisa el progreso de la compilación hasta que se complete.
4. En la ventana del explorador que muestra el portal de **Azure DevOps**, en el panel de navegación vertical, en la sección **Canalizaciones**, haz clic en **Versiones**, haz clic en la entrada que representa la versión y, en el panel **Selenium > Release-1**, haz clic en **Desarrollo**.
5. En el panel **Selenium > Release-1 > Desarrollo**, supervisa la implementación correspondiente.
6. Una vez iniciada la fase de **ejecución de pruebas de Selenium**, supervisa las pruebas del explorador web.
7. Una vez completada la versión, en el panel **Selenium > Release-1 > Desarrollo**, haz clic en la pestaña **Pruebas** para analizar los resultados de la prueba. Selecciona los filtros necesarios en la lista desplegable de la sección **Resultados** para ver las pruebas y su estado.

### Ejercicio 2: eliminación de los recursos del laboratorio de Azure

En este ejercicio, quitarás los recursos de Azure aprovisionados en este laboratorio para eliminar cargos inesperados.

>**Nota**: No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

#### Tarea 1: eliminar los recursos del laboratorio de Azure

En esta tarea, usarás Azure Cloud Shell para quitar los recursos de Azure aprovisionados en este laboratorio con el propósito de eliminar cargos innecesarios.

1. En Azure Portal, abra la sesión de shell de **Bash** en el panel **Cloud Shell**.
2. Ejecute el comando siguiente para enumerar todos los grupos de recursos que se han creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].name" --output tsv
    ```

3. Ejecute el comando siguiente para eliminar todos los grupos de recursos que ha creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro --nowait). Aunque podrá ejecutar otro comando de la CLI de Azure inmediatamente después en la misma sesión de Bash, los grupos de recursos tardarán unos minutos en quitarse.

## Revisar

Este laboratorio te enseñará a ejecutar casos de prueba de Selenium en una aplicación web de C# como parte de la canalización de versión de Azure DevOps.
