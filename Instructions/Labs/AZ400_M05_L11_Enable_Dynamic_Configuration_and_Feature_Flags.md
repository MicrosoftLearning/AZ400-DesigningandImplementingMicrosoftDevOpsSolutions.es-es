---
lab:
  title: Habilitación de marcas de características y configuración dinámica
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Habilitación de marcas de características y configuración dinámica

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps.](https://learn.microsoft.com/azure/devops/server/compatibility)

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

En este ejercicio, configurarás los requisitos previos para el laboratorio, que consta de un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de Azure DevOps **eShopOnWeb** que usarán varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asigna al proyecto el nombre **eShopOnWeb** y elige **Scrum** en la lista desplegable **Proceso de elemento de trabajo**. Haga clic en **Crear**.

#### Tarea 2: (omitir si ya la has completado) importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que usarán varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar**. En la ventana **Importar un repositorio de Git** pega la siguiente dirección URL https://github.com/MicrosoftLearning/eShopOnWeb.git y haz clic en **Importar**:

2. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **.azure** contiene la infraestructura de Bicep&ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - La carpeta **.github** contiene definiciones de flujo de trabajo de GitHub YAML.
    - La carpeta **src** contiene el sitio web .NET 7 que se usa en los escenarios de laboratorio.

#### Tarea 3: (omitir si ya la has completado) Establecer la rama principal como rama predeterminada

1. Ve a **Repos>Ramas**.
2. Mantén el puntero sobre la rama **main** y haz clic en los puntos suspensivos situados a la derecha de la columna.
3. Haz clic en **Establecer como rama predeterminada**.

### Ejercicio 1: (omitir si ya lo has completado) Importar y ejecutar canalizaciones de CI/CD

En este ejercicio, importarás y ejecutarás la canalización de CI, configurarás la conexión de servicio con la suscripción de Azure y, después, importarás y ejecutarás la canalización de CD.

#### Tarea 1: Importar y ejecutar la canalización de CI

Empecemos con la importación de la canalización de CI denominada [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Ve a **Canalizaciones>Canalizaciones**.
2. Haz clic en el botón **Crear canalización** (si no hay canalizaciones) o en el botón **Nueva canalización** (si ya hay canalizaciones creadas).
3. Selecciona **Git de Azure Repos (YAML)**.
4. Selecciona el repositorio **eShopOnWeb**.
5. Selecciona **Archivo YAML de Azure Pipelines existente**.
6. Selecciona el archivo **/.ado/eshoponweb-ci.yml** y luego haz clic en **Continuar**.
7. Haga clic en el botón **Run** (Ejecutar) para ejecutar la canalización.
8. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar de nombre o mover**. Asígnale el nombre **eshoponweb-ci** y haz clic en **Guardar**.

#### Tarea 2: Administrar la conexión del servicio

Puede crear una conexión desde Azure Pipelines a servicios externos y remotos para ejecutar las tareas de un trabajo.

En esta tarea, crearás una entidad de servicio mediante la CLI de Azure, que permitirá que Azure DevOps haga lo siguiente:

- Implementar recursos en tu suscripción de Azure
- Implementar la aplicación eShopOnWeb

> **Nota**: Si ya tienes una entidad de servicio, puedes continuar directamente con la próxima tarea.

Necesitarás una entidad de servicio para implementar recursos de Azure desde Azure Pipelines.

Azure Pipeline crea automáticamente una entidad de servicio cuando se conecta a una suscripción de Azure desde dentro de una definición de canalización o al crear una nueva conexión de servicio desde la página de configuración del proyecto (opción automática). También puedes crear manualmente la entidad de servicio desde el portal o mediante la CLI de Azure y volver a usarla en proyectos.

1. En el equipo del laboratorio, inicia un explorador web, ve a [**Azure Portal**](https://portal.azure.com) e inicia sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción de Azure que vas a usar en este laboratorio, así como el rol Administrador global en el inquilino de Microsoft Entra asociado a esta suscripción.
2. En Azure Portal, haz clic en el icono de **Cloud Shell**, situado inmediatamente a la derecha del cuadro de texto de búsqueda en la parte superior de la página.
3. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.

   >**Nota**: si es la primera vez que inicias **Cloud Shell** y aparece el mensaje **No tiene ningún almacenamiento montado**, selecciona la suscripción que usas en este laboratorio y haz clic en **Crear almacenamiento**.

4. En el símbolo del sistema de **Bash**, en el panel de **Cloud Shell**, ejecuta los siguientes comandos para recuperar los valores del atributo de id. de suscripción de Azure:

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Nota**: copia ambos valores en un archivo de texto. Los necesitará más adelante en este laboratorio.

5. En el símbolo del sistema de **Bash**, en el panel **Cloud Shell**, ejecuta el siguiente comando para crear una entidad de servicio:

    ```sh
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Nota**: el comando generará una salida JSON. Copie los resultados en un archivo de texto. Lo necesitará más adelante en este laboratorio.

6. Después, desde el equipo del laboratorio, abre un explorador web y ve al proyecto **eShopOnWeb** de Azure DevOps. Haz clic en **Configuración del proyecto>Conexiones de servicio (en Canalizaciones)** y en **Nueva conexión de servicio**.

7. En la hoja **Nueva conexión de servicio**, selecciona **Administrador de recursos de Azure** y luego **Siguiente** (quizá debas desplazarte hacia abajo).

8. Elige **Entidad de servicio (manual)** y haz clic en **Siguiente**.

9. Rellena los campos vacíos con la información recopilada durante los pasos anteriores:
    - Identificador y nombre de la suscripción
    - Identificador de entidad de servicio (o id. de cliente), clave (o contraseña) e id. de inquilino.
    - En **Nombre de conexión de servicio**, escribe **azure subs**.  Se hará referencia a este nombre en las canalizaciones de YAML cuando necesites una conexión de servicio de Azure DevOps para comunicarte con la suscripción de Azure.

10. Haz clic en **Verificar y guardar**.

#### Tarea 3: importar y ejecutar la canalización de CD

Vamos a importar la canalización de CD denominada [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Ve a **Canalizaciones>Canalizaciones**.
2. Haz clic en el botón **Nueva canalización**.
3. Selecciona **Git de Azure Repos (YAML)**.
4. Selecciona el repositorio **eShopOnWeb**.
5. Selecciona **Archivo YAML de Azure Pipelines existente**.
6. Selecciona el archivo **/.ado/eshoponweb-cd-webapp-code.yml** y haz clic en **Continuar**.
7. En la definición de canalización de YAML, personaliza lo siguiente:
   - **YOUR-SUBSCRIPTION-ID** con el identificador de tu suscripción de Azure.
   - **az400eshop-NAME**, reemplaza NAME para que sea único globalmente.
   - **AZ400-EWebShop-NAME** con el nombre del grupo de recursos definido antes en el laboratorio.

8. Haz clic en **Guardar y ejecutar** y espera a que la canalización se ejecute correctamente.

    > **Nota**: La implementación puede tardar unos minutos en completarse.

    La definición de CD consta de las siguientes tareas:
    - **Recursos**: está preparado para desencadenarse automáticamente en función de la finalización de la canalización de CI. También descarga el repositorio para el archivo bicep.
    - **AzureResourceManagerTemplateDeployment**: implementa la aplicación web de Azure mediante la plantilla de bicep.

9. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar de nombre o mover**. Asígnale el nombre **eshoponweb-cd-webapp-code** y haz clic en **Guardar**.

### Ejercicio 2: Administrar Azure App Configuration

En este ejercicio, crearás el recurso de App Configuration en Azure, habilitarás la identidad administrada y luego probarás la solución completa.

> **Nota**: Este ejercicio no requiere ninguna aptitud de codificación. El código del sitio web ya implementa las funciones de Azure App Configuration.

Si quieres saber cómo implementar esto en la aplicación, echa un vistazo a estos tutoriales: [Uso de la configuración dinámica en una aplicación de ASP.NET Core](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core) y [Administración de las marcas de características en Azure App Configuration](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags).

#### Tarea 1: Crear el grupo de recursos de App Configuration

1. En  Azure Portal, busca el servicio **Azure App Configuration**.
2. Haz clic en **Crear configuración de la aplicación** y selecciona:
    - Tu suscripción de Azure.
    - El grupo de recursos creado anteriormente (debe denominarse **AZ400-EWebShop-NAME**).
    - Ubicación.
    - Un nombre único como **appcs-NAME-REGION**, por ejemplo.
    - Selecciona el plan de tarifa **Gratuito**.
3. Selecciona **Revisar y crear** y luego **Crear**.
4. Después de crear el servicio App Configuration, ve a **Información general** y copia o guarda el valor del **Punto de conexión**.

#### Tarea 2: Habilitar la identidad administrada

1. Ve a la aplicación web implementada mediante la canalización (debe denominarse **az400-webapp-NAME**).
2. En la sección **Configuración**, haz clic en **Identidad** y luego cambia el estado a **Activado**; en la sección **Asignado por el sistema**, haz clic en **guardar > sí** y espera unos segundos a que finalice la operación.
3. Vuelve al servicio App Configuration y haz clic en **Control de acceso** y después en **Agregar asignación de roles**.
4. En la sección **Rol**, selecciona **Lector de datos de App Configuration**.
5. En la sección **Miembros**, marca la opción **Administrar identidad** y luego selecciona la identidad administrada de la aplicación web (deben tener el mismo nombre).
6. Haz clic en **Revisar y asignar**.

#### Tarea 3: Configurar la aplicación web

Para asegurarte de que el sitio web tenga acceso a App Configuration, debes actualizar su configuración.

1. Vuelve a tu aplicación web.
2. En la sección **Configuración**, haz clic en **Configuración**.
3. Agrega dos nuevas opciones de configuración de la aplicación:
    - Primera opción de configuración de la aplicación
        - **Nombre:** UseAppConfig
        - **Valor: true**
    - Segunda opción de configuración de la aplicación
        - **Nombre:** AppConfigEndpoint
        - **Valor:***el valor que has guardado o copiado desde el punto de conexión de App Configuration. Debería ser similar a https://appcs-NAME-REGION.azconfig.io*.

4. Haz clic en **Aceptar** y después en **Guardar**. Espera a que se actualice la configuración.
5. Ve a **Información general** y haz clic en **Examinar**.
6. En este paso, no verás ningún cambio en el sitio web, ya que App Configuration no contiene datos. Esto es lo que harás en las tareas siguientes.

#### Tarea 4: Probar la administración de la configuración

1. En el sitio web, selecciona **Visual Studio** en la lista desplegable **Marca** y haz clic en el botón de flecha (**>**).
2. Verás un mensaje que indica *"NO HAY RESULTADOS QUE COINCIDAN CON LA BÚSQUEDA".* El objetivo de este laboratorio es poder actualizar ese valor sin actualizar el código del sitio web ni volver a implementarlo.
3. Para probar esto, vuelve a App Configuration.
4. En la sección **Operations**, selecciona **Configuration Explorer**.
5. Haz clic en **Create > Key-value** y agrega lo siguiente:
    - **Key:** eShopWeb:Settings:NoResultsMessage
    - **Value:***escribe el mensaje personalizado*
6. Haz clic en **Apply** y vuelve a tu sitio web y actualiza la página.
7. Deberías ver el nuevo mensaje en lugar del valor predeterminado anterior.

Felicidades. En esta tarea, probaste el **Explorador de configuración** en Azure App Configuration.

#### Tarea 5: Probar la marca de características

Vamos a seguir probando el administrador de características.

1. Para probar esto, vuelve a App Configuration.
2. En la sección **Operaciones**, seleccione **Administrador de características**.
3. Haz clic en **Crear** y luego agrega lo siguiente:
    - **Enable feature flag:** activada
    - **Feature flag name:** SalesWeekend
4. Haz clic en **Apply** y vuelve a tu sitio web y actualiza la página.
5. Deberías ver una imagen con el texto "ALL T-SHIRTS ON SALE THIS WEEKEND".
6. Puedes deshabilitar esta característica en App Configuration y, luego, verías que la imagen desaparece.

Felicidades. En esta tarea, probaste el **Administrador de características** en Azure App Configuration.

### Ejercicio 3: eliminación de los recursos del laboratorio de Azure

En este ejercicio, quitarás los recursos de Azure aprovisionados en este laboratorio para eliminar cargos inesperados.

>**Nota**: No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

#### Tarea 1: eliminar los recursos del laboratorio de Azure

En esta tarea, usarás Azure Cloud Shell para quitar los recursos de Azure aprovisionados en este laboratorio con el propósito de eliminar cargos innecesarios.

1. En Azure Portal, abra la sesión de shell de **Bash** en el panel **Cloud Shell**.
2. Ejecute el comando siguiente para enumerar todos los grupos de recursos que se han creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].name" --output tsv
    ```

3. Ejecute el comando siguiente para eliminar todos los grupos de recursos que ha creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'AZ400-EWebShop-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro --nowait). Aunque podrá ejecutar otro comando de la CLI de Azure inmediatamente después en la misma sesión de Bash, los grupos de recursos tardarán unos minutos en quitarse.

## Revisar

En este laboratorio, aprendiste a habilitar dinámicamente la configuración y a administrar las marcas de características.
