---
lab:
  title: Habilitación de marcas de características y configuración dinámica
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Habilitación de marcas de características y configuración dinámica

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique una suscripción de Azure existente o cree una.

- Compruebe que tiene una cuenta de Microsoft o de Microsoft Entra con el rol Propietario o Colaborador en la suscripción a Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Introducción al laboratorio

[Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/overview) proporciona un servicio para administrar de manera centralizada la configuración y las marcas de características de la aplicación. Los programas modernos, especialmente los que se ejecutan en una nube, tienen por lo general muchos componentes distribuidos. La propagación de valores de configuración entre estos componentes puede conducir a errores difíciles de solucionar durante la implementación de la aplicación. Use App Configuration para almacenar toda la configuración de la aplicación y proteger sus accesos en un solo lugar.

## Objetivos

Después de completar este laboratorio, podrá:

- Habilite la configuración dinámica.
- Administre marcas de características.

## Tiempo estimado: 45 minutos

## Instrucciones

### Ejercicio 0: (omitir si se ha realizado) Configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y elige **Scrum** en la lista desplegable **Proceso del elemento de trabajo**. Haga clic en **Crear**.

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos > Archivos**, **Importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

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

### Ejercicio 1: (omitir si ya lo has completado) importación y ejecución de canalizaciones de CI/CD

En este ejercicio, importarás canalizaciones de CI/CD para compilar e implementar la aplicación eShopOnWeb. La canalización de CI ya está preparada para compilar la aplicación y ejecutar pruebas. La canalización de CD implementará la aplicación en una aplicación web de Azure.

#### Tarea 1: importación y ejecución de la canalización de CI

Empecemos importando la canalización de CI denominada [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Ve a **Canalizaciones > Canalizaciones**.
1. Haz clic en el botón **Crear canalización** (si no hay canalizaciones) o en **Nueva canalización** (si ya hay canalizaciones creadas).
1. Selecciona **GIT de Azure Repos (YAML)**.
1. Selecciona el repositorio **eShopOnWeb**.
1. Selecciona el **archivo YAML de Azure Pipelines existente**.
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-ci.yml**, y haga clic en **Continuar**.
1. Haga clic en el botón **Run** (Ejecutar) para ejecutar la canalización.
1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones > Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar el nombre/Quitar**. Asígnale el nombre **eshoponweb-ci** y haz clic en **Guardar**.

#### Tarea 2: Importación y ejecución de la canalización de CD

Vamos a importar la canalización de CD denominada [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.
1. Haga clic en el botón **Nueva canalización**
1. Selecciona **GIT de Azure Repos (YAML)**.
1. Selecciona el repositorio **eShopOnWeb**.
1. Selecciona el **archivo YAML de Azure Pipelines existente**.
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-cd-webapp-code.yml** y haga clic en **Continuar**.
1. En la definición de canalización de YAML, personaliza los siguientes elementos:
   - Reemplaza **YOUR-SUBSCRIPTION-ID** con el identificador de la suscripción a Azure.
   - En **az400eshop-NAME**, reemplaza NAME para que sea único globalmente.
   - Reemplaza **AZ400-EWebShop-NAME** con el nombre del grupo de recursos definido antes en el laboratorio.

1. Haz clic en **Guardar y ejecutar** y espera a que la canalización se ejecute correctamente.

    > **Nota**: Debes hacer clic en **Guardar y ejecutar** dos veces. Si la ventana Trabajos muestra un mensaje de permiso necesario, selecciona **Implementar** en la ventana Trabajos, selecciona **Ver** y, a continuación, **Permitir** dos veces para completar la ejecución de la canalización.

    > **Nota**: la implementación puede tardar unos minutos en completarse.

    La definición de CD consta de las siguientes tareas:
    - **Recursos**: está preparado para desencadenarse automáticamente cuando finalice la canalización de CI. También descarga el repositorio para el archivo bicep.
    - **AzureResourceManagerTemplateDeployment**: implementa la aplicación web de Azure mediante la plantilla de bicep.

1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones > Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar el nombre/Quitar**. Asígnale el nombre **eshoponweb-cd-webapp-code** y haz clic en **Guardar**.

### Ejercicio 2: administración de Azure App Configuration

En este ejercicio, crearás el recurso de App Configuration en Azure, habilitarás la identidad administrada y luego verificarás la solución completa.

> **Nota**: este ejercicio no requiere conocimientos de codificación. El código del sitio web ya implementa las funciones de Azure App Configuration.

Si quieres saber cómo implementar esto en la aplicación, echa un vistazo a estos tutoriales: [Uso de la configuración dinámica en una aplicación ASP.NET Core](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) y [Administración de marcas de características en Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Tarea 1: crear el grupo de recursos de App Configuration

1. En el Portal de Azure, busca el servicio **App Configuration**.
1. Haz clic en **Crear configuración de la aplicación** y selecciona:
    - Su suscripción de Azure.
    - El grupo de recursos creado anteriormente (debe denominarse **AZ400-EWebShop-NAME**).
    - Ubicación.
    - Un nombre único, como **appcs-NAME-REGION**.
    - Selecciona el nivel de precios **Gratuito**.
1. Selecciona **Revisar y crear** y, luego, **Crear**.
1. Después de crear el servicio App Configuration, ve a **Información general** y copia o guarda el valor del **punto de conexión**.

#### Tarea 2: habilitar la identidad administrada

1. Ve a la aplicación web implementada mediante la canalización (debe denominarse **az400-webapp-NAME**).
1. En la sección **Configuración**, haz clic en **Identidad** y luego cambia el estado a **Activado**; en la sección **Asignado por el sistema**, haz clic en **guardar > sí** y espera unos segundos a que finalice la operación.
1. Vuelve al servicio App Configuration y haz clic en **Control de acceso** y, después, en **Agregar asignación de roles**.
1. En la sección **Rol**, selecciona **Lector de datos de App Configuration**.
1. En la sección **Miembros**, marca **Administrar identidad** y haz clic en **+ Seleccionar miembros**. En el campo **Identidad administrada**, selecciona **App Service (1)**, selecciona el servicio de aplicaciones y haz clic en **Seleccionar**.
1. Haz clic en **Revisar y asignar** dos veces para completar la asignación de roles.

#### Tarea 3: configurar la aplicación web

Para asegurarte de que el sitio web tenga acceso a App Configuration, debes actualizar la configuración.

1. Accede a la aplicación web.
1. En la sección **Configuración**, haga clic en **Variables de entorno**.
1. Agrega dos nuevos ajustes de la aplicación:
    - Primer ajuste de la aplicación
        - **Nombre:** UseAppConfig
        - **Valor: true**
    - Segundo ajuste de la aplicación
        - **Nombre:** AppConfigEndpoint
        - **Valor:***el valor que has guardado o copiado desde el punto de conexión de App Configuration. Debería ser similar a <https://appcs-NAME-REGION.azconfig.io>*

1. Haga clic en **Aplicar** y, a continuación, en **Confirmar** y espere a que se actualice la configuración.
1. Ve a **Información general** y haz clic en **Examinar**.
1. En este paso, no verás ningún cambio en el sitio web, ya que App Configuration no tiene ningún dato. Esto es lo que harás en las tareas siguientes.

#### Tarea 4: probar la administración de configuración

1. En el sitio web, selecciona **Visual Studio** en la lista desplegable **Marca** y haz clic en el botón de flecha (**>**).
1. Verás el mensaje *"NO HAY RESULTADOS QUE COINCIDAN CON LA BÚSQUEDA".* El objetivo de este laboratorio es poder actualizar ese valor sin actualizar el código del sitio web ni volver a implementarlo.
1. Para probar esto, vuelve a App Configuration.
1. En la sección **Operaciones**, selecciona **Explorador de configuración**.
1. Haz clic en **Crear > Clave-valor** y agrega:
    - **Clave:** eShopWeb:Settings:NoResultsMessage
    - **Valor:***escribe un mensaje personalizado*
1. Haz clic en **Aplicar**, vuelve a tu sitio web y actualiza la página.
1. Deberías ver el mensaje nuevo en lugar del valor predeterminado anterior.

#### Tarea 5: probar la marca de características

Vamos a seguir probando el administrador de características.

1. Para probar esto, vuelve a App Configuration.
1. En la sección **Operaciones**, seleccione **Administrador de características**.
1. Haz clic en **Crear** y agrega:
    - **Habilitación de la marca de características:** activada
    - **Nombre de marca de características:** SalesWeekend
1. Haz clic en **Aplicar**, vuelve a tu sitio web y actualiza la página.
1. Deberías ver una imagen con el texto "TODAS LAS CAMISETAS CON DESCUENTO ESTE FIN DE SEMANA".
1. Puedes deshabilitar esta característica en App Configuration y, luego, verías que la imagen desaparece.

   > [!IMPORTANT]
   > Recuerda eliminar los recursos creados en Azure Portal para evitar cargos innecesarios. Asegúrate de deshabilitar la canalización **eshoponweb-cd-webapp-code** o volverá a crear un grupo de recursos eliminado y los recursos asociados después de la siguiente ejecución de **eshoponweb-ci**.

## Revisar

En este laboratorio, has aprendido a habilitar la configuración de manera dinámica y a administrar las marcas de características.
