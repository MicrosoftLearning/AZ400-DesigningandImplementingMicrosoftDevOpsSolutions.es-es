---
lab:
  title: Habilitación de marcas de características y configuración dinámica
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Habilitación de marcas de características y configuración dinámica

## Manual de laboratorio para alumnos

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

## Tiempo estimado: 60 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y elige **Scrum** en la lista desplegable **Proceso del elemento de trabajo**. Haga clic en **Crear**.

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

1. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - Definiciones de flujo de trabajo de GitHub del contenedor de carpetas **.github**.
    - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

#### Tarea 3: (omitir si ya la has completado) Establecer la rama principal como rama predeterminada

1. Ve a **Repos>Ramas**.
1. Mantén el puntero sobre la rama **main** y haz clic en los puntos suspensivos a la derecha de la columna.
1. Haz clic en **Establecer como rama predeterminada**.

### Ejercicio 1: (omitir si ya lo has completado) importación y ejecución de canalizaciones de CI/CD

En este ejercicio, importarás y ejecutarás la canalización de CI, configurarás la conexión de servicio con la suscripción de Azure y, después, importarás y ejecutarás la canalización de CD.

#### Tarea 1: importar y ejecutar la canalización de CI

Empecemos importando la canalización de CI denominada [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Ve a **Canalizaciones>Canalizaciones**.
1. Haz clic en el botón **Crear canalización** (si no hay canalizaciones) o en **Nueva canalización** (si ya hay canalizaciones creadas).
1. Selecciona **GIT de Azure Repos (YAML)**.
1. Selecciona el repositorio **eShopOnWeb**.
1. Selecciona el **archivo YAML de Azure Pipelines existente**.
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-ci.yml**, y haga clic en **Continuar**.
1. Haga clic en el botón **Run** (Ejecutar) para ejecutar la canalización.
1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar el nombre/Quitar**. Asígnale el nombre **eshoponweb-ci** y haz clic en **Guardar**.

#### Tarea 2: administrar la conexión de servicio

Puede crear una conexión desde Azure Pipelines a servicios externos y remotos para ejecutar las tareas de un trabajo.

En esta tarea, crearás una entidad de servicio mediante la CLI de Azure, que permitirá a Azure DevOps realizar lo siguiente:

- La implementación de recursos en una suscripción de Azure
- La implementación de la aplicación eShopOnWeb

> **Nota**: si ya tienes una entidad de servicio, puedes continuar directamente con la próxima tarea.

Necesitarás una entidad de servicio para implementar recursos de Azure desde Azure Pipelines.

Azure Pipeline crea automáticamente una entidad de servicio cuando se conecta a una suscripción de Azure desde dentro de una definición de canalización o al crear una nueva conexión de servicio desde la página de configuración del proyecto (opción automática). También puedes crear manualmente la entidad de servicio desde el portal o mediante la CLI de Azure, y volver a usarla en otros proyectos.

1. En el equipo del laboratorio, abre un explorador web, ve al [**Portal de Azure**](https://portal.azure.com) e inicia sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que vas a usar en este laboratorio, así como el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción.
1. En el portal de Azure portal, haz clic en el icono de **Cloud Shell**, situado inmediatamente a la derecha del cuadro de texto de búsqueda en la parte superior de la página
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.

   >**Nota**: si es la primera vez que inicias **Cloud Shell** y aparece el mensaje **No tienes ningún almacenamiento montado**, selecciona la suscripción que utilizas en este laboratorio y haz clic en **Crear almacenamiento**.

1. En el símbolo del sistema de **Bash**, en el panel de **Cloud Shell**, ejecuta los siguientes comandos para recuperar los valores del atributo de identificador de suscripción de Azure:

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Nota**: copia ambos valores en un archivo de texto. Los necesitará más adelante en este laboratorio.

1. En el símbolo del sistema de **Bash**, en el panel **Cloud Shell**, ejecuta el siguiente comando para crear una entidad de servicio:

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Nota**: el comando generará una salida JSON. Copie los resultados en un archivo de texto. Lo necesitará más adelante en este laboratorio.

1. Después, desde el equipo del laboratorio, abre un explorador web y ve al proyecto **eShopOnWeb** de Azure DevOps. Haz clic en **Configuración del proyecto>Conexiones de servicio (en Canalizaciones)** y en **Nueva conexión de servicio**.

1. En la hoja **Nueva conexión de servicio**, selecciona **Administrador de recursos de Azure** y luego **Siguiente** (quizá debas desplazarte hacia abajo).

1. Elige **Entidad de servicio (manual)** y haz clic en **Siguiente**.

1. Rellena los campos vacíos con la información recopilada durante los pasos anteriores:
    - Identificador y nombre de la suscripción
    - Identificador de entidad de servicio (o clientId), clave (o contraseña) y TenantId.
    - En **Nombre de conexión de servicio**, escribe **azure subs**. Se hará referencia a este nombre en las canalizaciones de YAML cuando necesites una conexión de servicio de Azure DevOps para comunicarte con la suscripción de Azure.

1. Haz clic en **Comprobar y guardar**.

#### Tarea 3: importar y ejecutar la canalización de CD

Vamos a importar la canalización de CD denominada [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Ve a **Canalizaciones>Canalizaciones**.
1. Haz clic en el botón **Nueva canalización**.
1. Selecciona **GIT de Azure Repos (YAML)**.
1. Selecciona el repositorio **eShopOnWeb**.
1. Selecciona el **archivo YAML de Azure Pipelines existente**.
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-cd-webapp-code.yml** y haga clic en **Continuar**.
1. En la definición de canalización de YAML, personaliza los siguientes elementos:
   - Reemplaza **YOUR-SUBSCRIPTION-ID** con el identificador de la suscripción a Azure.
   - En **az400eshop-NAME**, reemplaza NAME para que sea único globalmente.
   - Reemplaza **AZ400-EWebShop-NAME** con el nombre del grupo de recursos definido antes en el laboratorio.

1. Haz clic en **Guardar y ejecutar** y espera a que la canalización se ejecute correctamente.

    > **Nota**: la implementación puede tardar unos minutos en completarse.

    La definición de CD consta de las siguientes tareas:
    - **Recursos**: está preparado para desencadenarse automáticamente cuando finalice la canalización de CI. También descarga el repositorio para el archivo bicep.
    - **AzureResourceManagerTemplateDeployment**: implementa la aplicación web de Azure mediante la plantilla de bicep.

1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar el nombre/Quitar**. Asígnale el nombre **eshoponweb-cd-webapp-code** y haz clic en **Guardar**.

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
1. En la sección **Miembros**, marca la opción **Administrar identidad** y luego selecciona la identidad administrada de la aplicación web (deben tener el mismo nombre).
1. Haz clic en **Revisar y asignar**.

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

Felicidades. En esta tarea, has probado el **Explorador de configuración** en Azure App Configuration.

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

Felicidades. En esta tarea, has probado el **Administrador de características** en Azure App Configuration.

### Ejercicio 3: eliminación de los recursos del laboratorio de Azure

En este ejercicio, quitarás los recursos de Azure aprovisionados en este laboratorio para eliminar cargos inesperados.

>**Nota**: No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

#### Tarea 1: eliminar los recursos del laboratorio de Azure

En esta tarea, usarás Azure Cloud Shell para quitar los recursos de Azure aprovisionados en este laboratorio con el propósito de eliminar cargos innecesarios.

1. En Azure Portal, abra la sesión de shell de **Bash** en el panel **Cloud Shell**.
1. Ejecute el comando siguiente para enumerar todos los grupos de recursos que se han creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

1. Ejecute el comando siguiente para eliminar todos los grupos de recursos que ha creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro --nowait). Aunque podrá ejecutar otro comando de la CLI de Azure inmediatamente después en la misma sesión de Bash, los grupos de recursos tardarán unos minutos en quitarse.

## Revisar

En este laboratorio, has aprendido a habilitar la configuración de manera dinámica y a administrar las marcas de características.
