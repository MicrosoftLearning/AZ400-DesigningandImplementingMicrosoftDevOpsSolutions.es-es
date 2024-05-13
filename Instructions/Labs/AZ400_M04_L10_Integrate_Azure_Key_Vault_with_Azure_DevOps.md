---
lab:
  title: Integración de Azure Key Vault con Azure DevOps
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Integración de Azure Key Vault con Azure DevOps

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).
- Identifique una suscripción de Azure existente o cree una.

## Introducción al laboratorio

Azure Key Vault proporciona un almacenamiento y una administración seguros de los datos confidenciales, como claves, contraseñas y certificados. Azure Key Vault incluye compatibilidad con módulos de seguridad de hardware y una gama de algoritmos de cifrado y longitudes de clave. Haciendo uso de Azure Key Vault puedes minimizar la posibilidad de revelar datos confidenciales a través del código fuente, que es un error común cometido por los desarrolladores. El acceso a Azure Key Vault requiere una autenticación y autorización adecuadas, y admite permisos específicos para su contenido.

En este laboratorio, verás cómo puede integrar Azure Key Vault con una canalización de Azure Pipelines mediante los siguientes pasos:

- Crea un almacén de Azure Key Vault para almacenar una contraseña ACR como secreto.
- Crea una Entidad de servicio de Azure para proporcionar acceso a los secretos de Azure Key Vault.
- Configurar los permisos para permitir que la entidad de servicio lea el secreto.
- Configura la canalización para recuperar la contraseña de Azure Key Vault y pasarla a las tareas posteriores.

## Objetivos

Después de completar este laboratorio, podrá:

- Cree una entidad de servicio de Microsoft Entra.
- Cree un almacén de Azure Key Vault.

## Tiempo estimado: 40 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y deja los demás campos con los valores predeterminados. Haga clic en **Crear**.

    ![Crear proyecto](images/create-project.png)

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL https://github.com/MicrosoftLearning/eShopOnWeb.git y haz clic en **Importar**:

    ![Importar repositorio](images/import-repo.png)

1. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - Definiciones de flujo de trabajo de GitHub del contenedor de carpetas **.github**.
    - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

### Ejercicio 1: Configurar la canalización de CI para crear el contenedor eShopOnWeb.

Configura la canalización de YAML de CI para lo siguiente:

- Crear un Azure Container Registry para guardar las imágenes de los contenedores
- Uso de Docker Compose para compilar e insertar imágenes de contenedor **eshoppublicapi** y **eshopwebmvc**. Solo se implementará el contenedor **eshopwebmvc**.

#### Tarea 1: (omitir si has terminado) Crear una entidad de servicio principal

En esta tarea, crearás una entidad de servicio mediante la CLI de Azure, que permitirá que Azure DevOps haga lo siguiente:

- Implementar recursos en tu suscripción de Azure.
- Tener acceso de lectura en los secretos de Key Vault creados posteriormente.

> **Nota**: Si ya tienes una entidad de servicio, puedes continuar directamente con la siguiente tarea.

Necesitarás una entidad de servicio para implementar recursos de Azure desde Azure Pipelines. Dado que vamos a recuperar secretos en una canalización, es necesario conceder permiso al servicio al crear Azure Key Vault.

Azure Pipelines crea automáticamente una entidad de servicio cuando te conectas a una suscripción de Azure desde dentro de una definición de canalización o al crear una nueva conexión de servicio desde la página de configuración del proyecto (opción automática). También puedes crear manualmente la entidad de servicio desde el portal o mediante la CLI de Azure y volver a usarla en proyectos.

1. En el equipo del laboratorio, abre un explorador web, ve al [**Portal de Azure**](https://portal.azure.com) e inicia sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que vas a usar en este laboratorio, así como el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción.
1. En el portal de Azure portal, haz clic en el icono de **Cloud Shell**, situado inmediatamente a la derecha del cuadro de texto de búsqueda en la parte superior de la página
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.

   >**Nota**: si es la primera vez que inicias **Cloud Shell** y aparece el mensaje **No tienes ningún almacenamiento montado**, selecciona la suscripción que utilizas en este laboratorio y haz clic en **Crear almacenamiento**.

1. En el símbolo del sistema de **Bash**, en el panel **Cloud Shell**, ejecuta los siguientes comandos para recuperar los valores de la id. de suscripción de Azure y los atributos de la id. de suscripción:

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **Nota**: copia ambos valores en un archivo de texto. Los necesitará más adelante en este laboratorio.

1. En el símbolo del sistema **Bash**, en el panel de **Cloud Shell**, ejecuta el siguiente comando para crear una entidad de servicio (reemplaza **myServicePrincipalName** por cualquier cadena única de caracteres que consta de letras y dígitos) y **mySubscriptionID** por su subscriptionId de Azure:

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > **Nota**: el comando generará una salida JSON. Copie los resultados en un archivo de texto. Lo necesitará más adelante en este laboratorio.

1. Después, desde el equipo del laboratorio, abre un explorador web y ve al proyecto **eShopOnWeb** de Azure DevOps. Haz clic en **Configuración del proyecto>Conexiones de servicio (en Canalizaciones)** y en **Nueva conexión de servicio**.

    ![Nueva conexión de servicio](images/new-service-connection.png)

    > **Nota**: Si no hay conexiones de servicio creadas anteriormente en la página, el botón de creación de la conexión de servicio se encuentra en el centro de la página y tiene la etiqueta **Crear conexión de servicio**

1. En la hoja **Nueva conexión de servicio**, selecciona **Administrador de recursos de Azure** y luego **Siguiente** (quizá debas desplazarte hacia abajo).

1. A continuación, elija **Entidad de servicio (manual)** y haga clic en **Siguiente**.

1. Rellena los campos vacíos con la información recopilada durante los pasos anteriores:
    - Identificador y nombre de la suscripción.
    - Identificador de entidad de servicio (appId), clave de entidad de servicio (contraseña) e identificador de inquilino (inquilino).
    - En **Nombre de conexión de servicio**, escribe **azure subs**. Se hará referencia a este nombre en las canalizaciones de YAML cuando necesites una conexión de servicio de Azure DevOps para comunicarte con la suscripción de Azure.

    ![Conexión del servicio de Azure](images/azure-service-connection.png)

1. Haz clic en **Comprobar y guardar**.

#### Tarea 2: instalar y ejecutar la canalización de CI

En esta tarea, importarás una definición de canalización de CI YAML existente, la modificarás y ejecutarás. Crearás una nueva instancia de Azure Container Registry (ACR) y compilarás o publicarás las imágenes de contenedor eShopOnWeb.

1. En el equipo de laboratorio, abre un explorador web y ve al proyecto de Azure DevOps **eShopOnWeb**. Ve a **Canalizaciones>Canalizaciones** y haz clic en **Crear canalización** (o **Nueva canalización**).

1. En la ventana **¿Dónde está el código?**, selecciona **Azure Repos Git (YAML)** y selecciona el repositorio **eShopOnWeb**.

1. En la sección **Configurar**, selecciona **Archivo YAML de Azure Pipelines existente**. Seleccionar rama: **principal**, proporcione la siguiente ruta de acceso **/.ado/eshoponweb-ci-dockercompose.yml** y haga clic en **Continuar**.

    ![Selecciona Canalización](images/select-ci-container-compose.png)

1. En la definición de canalización de YAML, personalice el nombre del grupo de recursos reemplazando **NAME** en **AZ400-EWebShop-NAME** por un valor único y reemplace **YOUR-SUBSCRIPTION-ID** por su propio subscriptionId de Azure.

1. Haz clic en **Guardar y ejecutar** y espera a que la canalización se ejecute correctamente.

    > **Importante**: si ves el mensaje “Esta canalización necesita permisos para acceder a los recursos antes de que la ejecución pueda pasar de Docker Compose a ACI”, haz clic en Ver, Permitir y Permitir nuevamente. Esto es necesario para permitir que la canalización cree el recurso.

    > **Nota**: la compilación tardará algunos minutos en completarse. La canalización de compilación está formada por las siguientes tareas:
    - **AzureResourceManagerTemplateDeployment** usa **Bicep** para implementar una instancia de Azure Container Registry.
    - La tarea de **PowerShell** toma la salida de Bicep (servidor de inicio de sesión acr) y crea una variable de canalización.
    - La tarea **DockerCompose** compila e inserta las imágenes de contenedor para eShopOnWeb en Azure Container Registry.

1. La canalización tomará un nombre en función del nombre del proyecto. Permite **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar el nombre/Quitar**. Asígnale el nombre **eshoponweb-ci-dockercompose** y haz clic en **Guardar**.

1. Una vez finalizada la ejecución, en Azure Portal, abre el grupo de recursos definido previamente y verás la instancia de Azure Container Registry (ACR) con las imágenes de contenedor creadas, **eshoppublicapi** y **eshopwebmvc**. Solo usarás **eshopwebmvc** en la fase de implementación.

    ![Imágenes de contenedor en ACR](images/azure-container-registry.png)

1. Haga clic en **Claves de acceso**, habilite el **Usuario administrador** si aún no lo ha hecho y copie el valor de **contraseña**. Se usará en la siguiente tarea, ya que lo mantendremos como secreto en Azure Key Vault.

    ![Contraseña de ACR](images/acr-password.png)

#### Tarea 2: crear una instancia del Almacén de claves de Azure

En esta tarea, crearás una instancia del Almacén de claves de Azure mediante el portal de Azure.

En este escenario de laboratorio, tendremos una instancia de Azure Container (ACI) que extrae y ejecuta una imagen de contenedor almacenada en Azure Container Registry (ACR). Tenemos previsto almacenar la contraseña de ACR como un secreto en el almacén de claves.

1. En el portal de Azure, usa el cuadro de texto **Buscar recursos, servicios y documentos**, en la parte superior de la página de Azure Portal, escriba **Directiva** y presiona la tecla **Entrar**.
1. Selecciona la hoja **Almacenes de claves**, haz clic en  **Crear>Almacén de claves**.
1. En la pestaña **Aspectos básicos** de la hoja **Crear un Almacén de claves**, especifica la siguiente configuración y haz clic en **Siguiente**:

    | Configuración | Valor |
    | --- | --- |
    | Suscripción | nombre de la suscripción de Azure que usa en este laboratorio |
    | Resource group | el nombre de un nuevo grupo de recursos **AZ400-EWebShop-NAME** |
    | Nombre del almacén de claves | cualquier nombre válido único, como **ewebshop-kv-NAME** (reemplaza NAME) |
    | Region | una región de Azure más cercana a la ubicación del entorno de laboratorio |
    | Plan de tarifa | **Estándar** |
    | Días durante los cuales se conservarán los almacenes eliminados | **7**
           |
    | Protección de purgas | **Deshabilitación de la protección de purga** |

1. En la pestaña **Configuración de acceso** de la hoja **Crear un almacén de claves**, selecciona **Directiva de acceso del almacén** y, luego, en la sección **Directivas de acceso**, haz clic en **+ Crear** para configurar una nueva directiva.

    > **Nota**: Debes proteger el acceso a los almacenes de claves permitiendo el acceso únicamente a aplicaciones y usuarios autorizados. Para acceder a los datos del almacén, deberás facilitar permisos de lectura (Obtiene o enumera) a la entidad de servicio creada anteriormente que usarás para la autenticación en la canalización.

    1. En la hoja **Permisos**, debajo de **Permisos de secretos**, marca  los permisos **Obtener** y **enumerar**. Haga clic en **Siguiente**.
    2. En la hoja **Entidad principal**, busca la **entidad de servicio creada anteriormente**, ya sea mediante el identificador o el nombre especificados y selecciónala en la lista. Haz clic en **Siguiente**, **Siguiente**, **Crear** (directiva de acceso).
    3. En la hoja **Revisar y crear**, haz clic en **Crear**.

1. De nuevo en la hoja **Crear un almacén de claves**, haz clic en **Revisar y crear > Crear.**

    > **Nota**: Espera a que se aprovisione Azure Key Vault. Debería tardar menos de un minuto.

1. En la página **Se completó la implementación**, haz clic en **Ir al recurso**.
1. En la hoja Azure Key Vault (ewebshop-kv-NAME), en el menú vertical del lado izquierdo de la hoja, en la sección **Objetos**, haz clic en **Secretos**.
1. En la hoja **Secretos**, haz clic en **Generar e importar**.
1. En la hoja **Crear un secreto**, especifica las siguientes opciones de configuración (deja las demás con los valores predeterminados) y haz clic en **Crear**:

    | Configuración | Valor |
    | --- | --- |
    | Opciones de carga | **Manual** |
    | Nombre | **acr-secret** |
    | Value | Contraseña de acceso de ACR copiada en la tarea anterior |

#### Tarea 3: Creación de un grupo de variables conectado a Azure Key Vault

En esta tarea, creará un grupo de variables en Azure DevOps que recuperará el secreto de contraseña de ACR de Key Vault mediante el servicio Conexión de servicio (entidad de servicio).

1. En el equipo de laboratorio, inicia un explorador web y navega hasta el proyecto de Azure DevOps **eShopOnWeb**.

1. En el panel de navegación vertical del portal de Azure DevOps, selecciona **Canalizaciones>Biblioteca**. Haz clic en **+ Grupo de variables**.

1. En el panel **Nuevo grupo de variables**, configura las opciones siguientes:

    | Configuración | Valor |
    | --- | --- |
    | Nombre del grupo de variables | **eshopweb-vg** |
    | Vincular secretos desde un Almacén de claves de Azure | **enable** |
    | Suscripción de Azure | **Conexiones disponibles del servicio de Azure > Suscripciones de Azure** |
    | Nombre del almacén de claves | Nombre del almacén de claves|

1. En **Variables**, haz clic en **+ Agregar** y selecciona el secreto **acr-secret**. Haga clic en **Aceptar**.
1. Haga clic en **Guardar**.

    ![Creación de grupo de variables](images/vg-create.png)

#### Tarea 4: configurar la canalización de CD para implementar el contenedor en instancias de Azure Container (ACI)

En esta tarea, importarás una canalización de CD, la personalizarás y ejecutarás para implementar la imagen de contenedor creada antes en una instancia de contenedor de Azure.

1. En el equipo de laboratorio, abre un explorador web y ve al proyecto de Azure DevOps **eShopOnWeb**. Ve a **Canalizaciones>Canalizaciones** y haz clic en **Nueva canalización**.

1. En la ventana **¿Dónde está el código?**, selecciona **Azure Repos Git (YAML)** y selecciona el repositorio **eShopOnWeb**.

1. En la sección **Configurar**, selecciona **Archivo YAML de Azure Pipelines existente**. Seleccionar rama: **principal**, proporcione la siguiente ruta de acceso **/.ado/eshoponweb-cd-aci.yml** y haga clic en **Continuar**.

1. En la definición de canalización de YAML, personaliza los siguientes elementos:

    - Reemplaza **YOUR-SUBSCRIPTION-ID** con el identificador de la suscripción a Azure.
    - En **az400eshop-NAME**, reemplaza NAME para que sea único globalmente.
    - ** YOUR-ACR.azurecr.io** y **ACR-USERNAME** con el servidor de inicio de sesión de ACR (ambos necesitan el nombre de ACR y se pueden revisar en ACR>Claves de acceso).
    - Reemplaza **AZ400-EWebShop-NAME** con el nombre del grupo de recursos definido antes en el laboratorio.

1. Haz clic en **Guardar y ejecutar**.
1. Abre la canalización y espera a que se ejecute correctamente.

    > **Importante**: si ves el mensaje “Esta canalización necesita permisos para acceder a los recursos antes de que la ejecución pueda pasar de Docker Compose a ACI”, haz clic en Ver, Permitir y Permitir nuevamente. Esto es necesario para permitir que la canalización cree el recurso.

    > **Nota**: la implementación puede tardar unos minutos en completarse. La definición de CD consta de las siguientes tareas:
    - **Recursos**: están preparados para desencadenarse automáticamente en función de la finalización de la canalización de CI. También descarga el repositorio para el archivo Bicep.
    - **Variables (para la fase de implementación)** se conecta con el grupo de variables para utilizar el secreto del Almacén de claves de Azure, **acr-secret**
    - **AzureResourceManagerTemplateDeployment** implementa la instancia de Azure Container (ACI) mediante la plantilla de bicep y proporciona los parámetros de inicio de sesión de ACR para permitir que ACI descargue la imagen de contenedor creada anteriormente desde Azure Container Registry (ACR).

1. La canalización tomará un nombre en función del nombre del proyecto. Permite **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar el nombre/Quitar**. Asígnale el nombre **eshoponweb-cd-aci** y haz clic en **Guardar**.

### Ejercicio 2: eliminación de los recursos del laboratorio de Azure

En este ejercicio, quitarás los recursos de Azure aprovisionados en este laboratorio para eliminar cargos inesperados.

>**Nota**: No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

#### Tarea 1: eliminar los recursos del laboratorio de Azure

En esta tarea, usarás Azure Cloud Shell para quitar los recursos de Azure aprovisionados en este laboratorio con el propósito de eliminar cargos innecesarios.

1. Abre el grupo de recursos en el portal de Azure y haz clic en **Eliminar grupo de recursos**.

## Revisar

En este laboratorio, integrarás el Almacén de claves de Azure con una canalización de Azure DevOps siguiendo estos pasos:

- Has creado una entidad de servicio de Azure para proporcionar acceso a un secreto del Almacén de claves de Azure y autenticar la implementación en Azure desde Azure DevOps.
- Ejecuta dos canalizaciones de YAML importadas desde un repositorio de Git.
- Se ha configurado una canalización para recuperar la contraseña del Almacén de claves de Azure mediante un grupo de variables y usarla en tareas posteriores.
