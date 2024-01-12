---
lab:
  title: Creación de un panel de versión
  module: 'Module 04: Design and implement a release strategy'
---

# Creación de un panel de versión

# Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique una suscripción de Azure existente o cree una.

- Comprueba que tienes una cuenta de Microsoft o de Azure AD con el rol Propietario en la suscripción a Azure. Para obtener información detallada, vea [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal).

## Introducción al laboratorio

En este laboratorio, se le guiará en la creación de un panel de versión y el uso de la API de REST para recuperar los datos de versión de Azure DevOps, que puede poner a disposición de las aplicaciones o paneles personalizados.

El laboratorio usa el recurso Azure DevOps Starter, que crea automáticamente un proyecto de Azure DevOps que compila e implementa una aplicación en Azure.

## Objetivos

Después de completar este laboratorio, podrá:

- Crear un panel de versión
- Usar la API de REST para consultar la información de la versión

## Tiempo estimado: 45 minutos

## Instrucciones

### Ejercicio 1: creación de un panel de versión

En este ejercicio, crearás un panel de versión en una organización de Azure DevOps.

#### Tarea 1: crear un recurso de Azure DevOps Starter

En esta tarea, crearás un recurso de Azure DevOps Starter en tu suscripción de Azure. Esto creará automáticamente un proyecto correspondiente en la organización de Azure DevOps.

1. En el equipo del laboratorio, inicia un explorador web, ve al [**Portal de Azure **](https://portal.azure.com) e inicia sesión con las credenciales de una cuenta de usuario con el rol Propietario o Colaborador en la suscripción de Azure que usarás en este laboratorio.
1. En el Portal de Azure, busca y selecciona el tipo de recurso **DevOps Starter** y, en la hoja **DevOps Starter**, haz clic en **+ Crear**.
1. En la hoja **DevOps Starter**, en el panel **Iniciar de cero con una nueva aplicación**, selecciona el mosaico **.NET** y, en la parte superior, junto a **Configuración de DevOps Starter con GitHub**, cambia la configuración, haz clic **aquí** y selecciona **Azure DevOps**, **Listo** y **Siguiente: Marco >**.
1. En la hoja **DevOps Starter**, en el panel **Elegir un marco de aplicación**, selecciona el mosaico **asp.NET<nolink> Core**, mueve el control deslizante **Agregar una base de datos** a la posición de **Activado** y haz clic en **Siguiente: Servicio >**.
1. En la hoja **DevOps Starter**, en el panel **Seleccionar un servicio de Azure para implementar la aplicación**, asegúrate de que el mosaico **Aplicación web de Windows** esté seleccionado y haz clic en **Siguiente: Crear >**.
1. En la hoja **DevOps Starter**, en el panel **Casi hemos terminado**, especifica la siguiente configuración:

    | Configuración | Valor |
    | ------- | ----- |
    | Nombre de proyecto | **Creación de un panel de versión** |
    | Organización de Azure DevOps | el nombre de la organización de Azure DevOps que usarás en este ejercicio |
    | Subscription | nombre de la suscripción de Azure que usa en este laboratorio |
    | Nombre de aplicación web | cualquier nombre único global de entre 2 y 60 caracteres, que conste de letras, números y guiones, y que empiece y termine con una letra o un número |
    | Location | el nombre de la región de Azure en la que vas a implementar una aplicación web de Azure y una base de datos de Azure SQL |

1. En la hoja **DevOps Starter**, en el panel **Casi hemos terminado**, haz clic en **Configuración adicional**.
1. En el panel **Configuración adicional**, especifica la siguiente configuración y haz clic en **Aceptar**.

    | Configuración | Value |
    | ------- | ----- |
    | Resource group | **az400m10l02-rg** |
    | Plan de tarifa | **F1 gratis** |
    | Ubicación de Application Insights | el nombre de la misma región de Azure que has elegido para la ubicación de la aplicación web de Azure |
    | Nombre del servidor | cualquier nombre único global de entre 3 y 63 caracteres, que conste de letras, números y guiones, y que empiece y termine con una letra o un número. |
    | Escriba el nombre de usuario | **dbadmin** |
    | Location | el nombre de la misma región de Azure que has elegido para la ubicación de la aplicación web de Azure |
    | Nombre de la base de datos | **az400m10l02-db** |

1. Cuando vuelvas a la hoja **DevOps Starter**, en el panel **Casi hermos terminado**, haz clic en **Listo** y después en **Revisar y crear**.

    > **Nota**: Espere a que la implementación se complete. El aprovisionamiento del recurso **DevOps Starter** debería tardar unos 2 minutos.

1. Una vez que recibas la confirmación de que se aprovisionó el recurso DevOps Starter, haz clic en el botón **Ir al recurso**. Esto te redirigirá a la hoja DevOps Starter.
1. En la hoja DevOps Starter, haz un seguimiento del progreso de la canalización de CI/CD hasta que finalice.

    > **Nota**: la creación de la aplicación web de Azure correspondiente y la base de datos de Azure SQL puede tardar unos 5 minutos. El proceso crea automáticamente un proyecto de Azure DevOps que incluye un repositorio listo para implementar, así como las canalizaciones de compilación y versión. Los recursos de Azure se crean como parte de la canalización de implementación desencadenada automáticamente.

#### Tarea 2: crear versiones de Azure DevOps

En esta tarea, crearás varias versiones de Azure DevOps, incluida una que provocará un error en la implementación.

1. En el explorador web que muestra el Portal de Azure, en la página DevOps Starter, en la barra de herramientas, haz clic en **Página principal del proyecto**. Se abrirá automáticamente otra pestaña del explorador que muestra el proyecto **Creación de un panel de versión** en el portal de Azure Devops. Si debes iniciar sesión, usa las credenciales de la organización de Azure DevOps.

    > **Nota**: en primer lugar, crearás una nueva versión que se implementará correctamente.

1. En el portal de Azure DevOps, en el menú vertical del lado izquierdo, haz clic en **Repos**; en la lista de carpetas del repositorio, ve a la carpeta **Aplicación\\aspnet-core-dotnet-core\\Páginas** y haz clic en la entrada **Index.cshtml**.
1. En el panel **Index.cshtml**, haz clic en **Editar**; en la línea **20**, reemplaza `<div class="description line-2"> Your ASP.NET Core app is up and running on Azure</div>` con `<div class="description line-2"> Your ASP.NET Core app v1.1 is up and running on Azure</div>` y haz clic en **Confirmar**; y en el panel **Confirmar**, haz clic en **Confirmar** nuevamente. Esto desencadenará automáticamente la canalización de compilación.
1. En el portal de Azure DevOps, en el panel de navegación vertical del lado izquierdo, haz clic en **Canalizaciones**.
1. En la pestaña **Recientes** del panel **Canalizaciones**, haz clic en la entrada **az400m10l02-CI**; en la pestaña **Ejecuciones** del panel **az400m10l02-CI**, selecciona la ejecución más reciente, en la pestaña **Resumen** de la ejecución, en la sección **Trabajos**, haz clic en **Compilar** y supervisa el trabajo hasta que finalice.
1. Una vez finalizado el trabajo, en el portal de Azure DevOps, en el panel de navegación vertical del lado izquierdo, en la sección **Canalizaciones**, haz clic en **Versiones**.
1. En el panel **az400m10l02 - CD**, en la pestaña **Versiones**, haz clic en la entrada **Release-2**, en la pestaña **Canalización** del panel **Release-2**, haz clic en la etapa de **desarrollo** y en el panel de **desarrollo**, haz clic en **Ver registros** y supervisa el progreso de la implementación hasta que finalice.

    > **Nota**: ahora crearás una nueva versión que producirá un error en la implementación. El error se debe a la prueba de ensamblados integrados, que considera que el cambio asociado a la nueva versión no es válido.

1. En el portal de Azure DevOps, en el menú vertical del lado izquierdo, haz clic en **Repos**; en la lista de carpetas del repositorio, ve a la carpeta **Aplicación\\aspnet-core-dotnet-core\\Páginas** y haz clic en la entrada **Index.cshtml**.
1. En el panel **Index.cshtml**, haz clic en **Editar**. En la línea **4**, reemplaza `ViewData["Title"] = "Home Page - ASP.NET Core";` con `ViewData["Title"] = "Home Page v1.2 - ASP.NET Core";`. Haga clic en **Confirmar**. En el panel **Confirmar**, haz clic en **Confirmar** nuevamente. Esto desencadenará automáticamente la canalización de compilación.
1. En el portal de Azure DevOps, en el panel de navegación vertical del lado izquierdo, haz clic en **Canalizaciones**.
1. En la pestaña **Recientes** del panel **Canalizaciones**, haz clic en la entrada **az400m10l02-CI**; en la pestaña **Ejecuciones** del panel **az400m10l02-CI**, selecciona la ejecución más reciente, en la pestaña **Resumen** de la ejecución, en la sección **Trabajos**, haz clic en **Compilar** y supervisa el trabajo hasta que finalice.
1. Una vez finalizado el trabajo, en el portal de Azure DevOps, en el panel de navegación vertical del lado izquierdo, en la sección **Canalizaciones**, haz clic en **Versiones**.
1. En el panel **az400m10l02 - CD**, en la pestaña **Versiones**, haz clic en la entrada **Release-3**; en la pestaña **Canalización** del panel **Release-3**, hazclic en la etapa de **desarrollo** y, en el panel de **desarrollo**, haz clic en **Ver registros** y supervisa el progreso de la implementación hasta que se produzca el error durante la etapa **Ensamblados de prueba**.

#### Tarea 3: crear un panel de versión de Azure DevOps

En esta tarea, crearás un panel y le agregarás widgets relacionados con la versión.

1. En el portal de Azure DevOps, en el menú vertical del lado izquierdo, haz clic en **Información general**; en la sección **Información general**, haz clic en **Paneles** y después en **Agregar un widget**.
1. En el panel **Agregar widget**, desplázate hacia abajo por la lista de widgets, selecciona la entrada **Estado de implementación** y haz clic en **Agregar**.
1. Usa el procedimiento descrito en el paso anterior para agregar los widgets **Detalles del estado de la versión**, **Información general del estado de la versión** e **Información general de canalización de la versión**.
    > **Nota**: instala **Detalles del estado de la versión** e **Información general del estado de la versión** del marketplace [Estado del proyecto del equipo](https://marketplace.visualstudio.com/items?itemName=ms-devlabs.TeamProjectHealth)
1. Usa el mouse para arrastrar la **Información general de canalización de versión** a la derecha del widget **Estado de implementación** para evitar la necesidad de desplazarse verticalmente por el panel y hacer clic en **Edición finalizada**.
1. Cuando vuelvas al panel, en el rectángulo que representa el widget **Estado de implementación**, haz clic en **Configurar widget**.
1. En el panel **Configuración**, especifica las opciones de configuración siguientes (deja las demás con los valores predeterminados) y haz clic en **Guardar**:

    | Configuración | Valor |
    | ------- | ----- |
    | Canalización de compilación | **az400m10l02 - CI** |
    | Canalizaciones de versión vinculadas | **az400m10l02 - CD; az400m10l02 - CD\dev** |

1. Cuando vuelvas al panel, pasa el puntero sobre la esquina superior derecha del rectángulo que representa el widget **Información general del estado de la versión** para mostrar el signo de puntos suspensivos del menú **Más acciones**; hazle clic y haz clic en **Configurar** en el menú desplegable.  
1. En el panel **Configuración**, especifica las opciones de configuración siguientes (deja las demás con los valores predeterminados) y haz clic en **Guardar**:

    | Configuración | Valor |
    | ------- | ----- |
    | Selecciona las definiciones de la versión | **az400m10l02 - CD** |

1. Cuando vuelvas al panel, pasa el puntero sobre la esquina superior derecha del rectángulo correspondiente al widget **Detalles del estado de la versión** para ver el signo de puntos suspensivos del menú **Más acciones**; hazle clic y haz clic en **Configurar** en el menú desplegable.  
1. En el panel **Configuración**, especifica las opciones de configuración siguientes (deja las demás con los valores predeterminados) y haz clic en **Guardar**:

    | Configuración | Valor |
    | ------- | ----- |
    | Definición | **az400m10l02 - CD** |

1. Cuando vuelvas al panel, pasa el puntero sobre la esquina superior derecha del rectángulo correspondiente al widget **Información general de canalización de la versión** para ver el signo de puntos suspensivos del menú **Más acciones**; hazle clic y haz clic en **Configurar** en el menú desplegable.  
1. En el panel **Configuración**, especifica las opciones de configuración siguientes (deja las demás con los valores predeterminados) y haz clic en **Guardar**:

    | Configuración | Valor |
    | ------- | ----- |
    | Canalización de versión | **az400m10l02 - CD** |

1. Cuando vuelvas al panel, haz clic en **Actualizar** para actualizar el contenido que muestran los widgets.

    > **Nota**: los vínculos de los widgets permiten navegar directamente a los paneles correspondientes en el portal de Azure DevOps.

### Ejercicio 2: consulta de información de la versión a través de la API de REST

En este ejercicio, consultarás la información de la versión a través de la API de REST mediante Postman.

#### Tarea 1: generar un token de acceso personal de Azure DevOps

En esta tarea, generarás un token de acceso personal de Azure DevOps que se usará para autenticar desde la aplicación Postman que instalarás en la siguiente tarea de este ejercicio.

1. En el equipo del laboratorio, en la ventana del explorador web que muestra el portal de Azure DevOps, en la esquina superior derecha de la página de Azure DevOps, haz clic en el icono **Configuración de usuario**; en el menú desplegable, haz clic en **Tokens de acceso personal** y en el panel **Tokens de acceso personal**, haz clic en **+ Nuevo token**.
1. En el panel **Crear un nuevo token de acceso personal**, haz clic en el vínculo **Mostrar todos los ámbitos**, especifica la siguiente configuración y haz clic en **Crear** (deja las demás opciones con sus valores predeterminados):

    | Configuración | Value |
    | --- | --- |
    | Nombre | **Laboratorio de creación de un panel de versión** |
    | Ámbito | **Versión** |
    | Permisos | **Leer** |
    | Ámbito | **Compilar** |
    | Permisos | **Leer** |

1. En el panel **Correcto**, copia el valor del token de acceso personal en el portapapeles.

    > **Nota**: asegúrate de registrar el valor del token. No podrás recuperarlo cuando cierres este panel.

1. En el panel **Correcto**, haz clic en **Cerrar**.

#### Tarea 2: consultar información de la versión a través de la API de REST mediante Postman

En esta tarea, consultarás la información de la versión a través de la API de REST mediante Postman.

1. En el equipo del laboratorio, abre un explorador web y ve a la [página de descarga de Postman](https://www.postman.com/downloads/), haz clic en el botón **Descargar la aplicación**. En el menú desplegable, haz clic en **Windows 64 bits**. Haz clic en el archivo descargado y ejecuta la instalación. Una vez finalizada la instalación, la aplicación de escritorio de Postman se iniciará automáticamente.
1. En el panel **Crear cuenta de Postman**, proporciona tu dirección de correo electrónico, un nombre de usuario y una contraseña, y haz clic en **Crear cuenta gratuita**.

    > **Nota**: recibirás un correo electrónico de Postman para activar tu cuenta y completar el proceso de aprovisionamiento de cuentas.

1. Una vez que hayas iniciado sesión, en la ventana de la aplicación de escritorio de Postman, en la esquina superior izquierda, haz clic en **+Nuevo**; en el panel **Crear nuevo**, haz clic en **Solicitar**. En el panel **GUARDAR SOLICITUD**, en el cuadro de texto **Nombre de la solicitud**, escribe **Get-Releases** y haz clic en **+ Crear colección**. En el cuadro de texto **Nombre de la colección**, escribe **Azure DevOps az400m10l02,** haz clic en la marca de verificación a la derecha y después haz clic en **Guardar en consultas az400m10l02 de Azure DevOps**.
1. Abre otra ventana del explorador web y ve a la [página **Versiones - Lista** de Microsoft Docs](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/releases/list?view=azure-devops-rest-6.0) y revisa su contenido.
1. Vuelve a la aplicación de escritorio de Postman. En el panel Launchpad de la sección superior derecha de la ventana de la aplicación, haz clic en el encabezado de pestaña **Autorización**. En la lista desplegable **TIPO**, selecciona la entrada **Autenticación básica** y, en el cuadro de texto **Contraseña**, pega el valor del token que has copiado en la tarea anterior (no establezcas el valor del cuadro de texto **Nombre de usuario**).
1. En el panel Launchpad de la sección superior derecha de la ventana de la aplicación, asegúrate de que **GET** aparezca en la lista desplegable. En el cuadro de texto **Escribir dirección URL de solicitud**, escribe lo siguiente y haz clic en **Enviar** (reemplaza el valor de `<organization_name>` por el nombre de la organización de Azure DevOps) para enumerar todas las versiones:

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/releases?api-version=6.0
    ```

1. Revisa la salida que aparece en la pestaña **Cuerpo** de la sección inferior derecha de la ventana de la aplicación y comprueba que tenga la lista de versiones creadas en el ejercicio anterior de este laboratorio.
1. Accede a la ventana del explorador web que muestra el contenido de Microsoft Docs, ve a [la página **Implementaciones - Lista** de Microsoft Docs](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/deployments/list?view=azure-devops-rest-6.0) y revisa su contenido.
1. En el panel Launchpad de la sección superior derecha de la ventana de la aplicación, asegúrate de que **GET** aparezca en la lista desplegable. En el cuadro de texto **Escribir dirección URL de solicitud**, escribe lo siguiente y haz clic en **Enviar** (reemplaza el valor de `<organization_name>` por el nombre de la organización de Azure DevOps) para enumerar todas las implementaciones:

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?api-version=6.0
    ```

1. Revisa la salida que aparece en la pestaña **Cuerpo** de la sección inferior derecha de la ventana de la aplicación y comprueba que tenga la lista de las implementaciones que has iniciado en el ejercicio anterior de este laboratorio.
1. En el panel Launchpad de la sección superior derecha de la ventana de la aplicación, asegúrate de que **GET** aparezca en la lista desplegable. En el cuadro de texto **Escribir dirección URL de solicitud**, escribe lo siguiente y haz clic en **Enviar** (reemplaza el valor de `<organization_name>` por el nombre de la organización de Azure DevOps) para enumerar todas las implementaciones:

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?DeploymentStatus=failed&api-version=6.0
    ```

1. Revisa la salida que aparece en la pestaña **Cuerpo** de la sección inferior derecha de la ventana de la aplicación y comprueba que solo tenga la implementación con errores que has iniciado en el ejercicio anterior de este laboratorio.

### Ejercicio 3: eliminación de los recursos del laboratorio de Azure

En este ejercicio, quitarás los recursos de Azure aprovisionados en este laboratorio para eliminar cargos inesperados.

>**Nota**: No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

#### Tarea 1: eliminar los recursos del laboratorio de Azure

En esta tarea, usarás Azure Cloud Shell para quitar los recursos de Azure aprovisionados en este laboratorio con el propósito de eliminar cargos innecesarios.

1. En Azure Portal, abra la sesión de shell de **Bash** en el panel **Cloud Shell**.
1. Ejecute el comando siguiente para enumerar todos los grupos de recursos que se han creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].name" --output tsv
    ```

1. Ejecute el comando siguiente para eliminar todos los grupos de recursos que ha creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro --nowait). Aunque podrá ejecutar otro comando de la CLI de Azure inmediatamente después en la misma sesión de Bash, los grupos de recursos tardarán unos minutos en quitarse.

## Revisar

En este laboratorio, has aprendido a crear y configurar el panel de versión y a usar la API de REST para recuperar los datos de la versión de Azure DevOps.
