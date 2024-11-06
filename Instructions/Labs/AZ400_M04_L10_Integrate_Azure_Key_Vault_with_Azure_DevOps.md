---
lab:
  title: Integración de Azure Key Vault con Azure DevOps
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# Integración de Azure Key Vault con Azure DevOps

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).
- Identifique una suscripción de Azure existente o cree una.

## Introducción al laboratorio

Azure Key Vault proporciona un almacenamiento y una administración seguros de los datos confidenciales, como claves, contraseñas y certificados. Azure Key Vault incluye compatibilidad con módulos de seguridad de hardware y una gama de algoritmos de cifrado y longitudes de clave. Haciendo uso de Azure Key Vault puedes minimizar la posibilidad de revelar datos confidenciales a través del código fuente, que es un error común cometido por los desarrolladores. El acceso a Azure Key Vault requiere una autenticación y autorización adecuadas, y admite permisos específicos para su contenido.

En este laboratorio, verás cómo puede integrar Azure Key Vault con una canalización de Azure Pipelines mediante los siguientes pasos:

- Crea un almacén de Azure Key Vault para almacenar una contraseña ACR como secreto.
- Proporciona acceso a secretos en Azure Key Vault.
- Configura permisos para leer el secreto.
- Configura la canalización para recuperar la contraseña de Azure Key Vault y pasarla a las tareas posteriores.

## Objetivos

Después de completar este laboratorio, podrá:

- Cree un almacén de claves de Azure Key Vault.
- Recuperar un secreto de Azure Key Vault en una canalización de Azure DevOps.
- Usar el secreto en una tarea posterior de la canalización.
- Implementar una imagen de contenedor en Azure Container Instance (ACI) mediante el secreto.

## Tiempo estimado: 40 minutos

## Instrucciones

### Ejercicio 0: (omitir si se ha realizado) Configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y deja los demás campos con los valores predeterminados. Haga clic en **Crear**.

    ![Captura de pantalla del panel Crear nuevo proyecto.](images/create-project.png)

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos > Archivos**, **Importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

    ![Captura de pantalla del Importar panel de repositorio.](images/import-repo.png)

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

### Ejercicio 1: Configurar la canalización de CI para crear el contenedor eShopOnWeb.

En este ejercicio, crearás una canalización de CI que compila e inserta las imágenes de contenedor eShopOnWeb en una instancia de Azure Container Registry (ACR). La canalización usará Docker Compose para compilar las imágenes e insertarlas en ACR.

#### Tarea 1: instalar y ejecutar la canalización de CI

En esta tarea, importarás una definición de canalización de CI YAML existente, la modificarás y ejecutarás. Crearás una nueva instancia de Azure Container Registry (ACR) y compilarás o publicarás las imágenes de contenedor eShopOnWeb.

1. En el equipo de laboratorio, abre un explorador web y ve al proyecto de Azure DevOps **eShopOnWeb**. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)** y haga clic en **Create Pipeline (Crear canalización)** o **New Pipeline (Nueva canalización)**.

1. En la ventana **Where is your code? (¿Dónde está el código?)**, seleccione **Git de Azure Repos (YAML)** y seleccione el repositorio **eShopOnWeb**.

1. En la sección **Configurar**, selecciona **Archivo YAML de Azure Pipelines existente**. Seleccionar rama: **principal**, proporcione la siguiente ruta de acceso **/.ado/eshoponweb-ci-dockercompose.yml** y haga clic en **Continuar**.

    ![Captura de pantalla de la canalización existente de YAML.](images/select-ci-container-compose.png)

1. En la definición de canalización de YAML, personalice el nombre del grupo de recursos reemplazando **NAME** en **AZ400-EWebShop-NAME** por un valor único y reemplace **YOUR-SUBSCRIPTION-ID** por su propio subscriptionId de Azure.

1. Haz clic en **Guardar y ejecutar** y espera a que la canalización se ejecute correctamente.

    > **Importante**: si ves el mensaje “Esta canalización necesita permisos para acceder a los recursos antes de que la ejecución pueda pasar de Docker Compose a ACI”, haz clic en Ver, Permitir y Permitir nuevamente. Esto es necesario para permitir que la canalización cree el recurso.

    > **Nota**: la compilación tardará algunos minutos en completarse. La canalización de compilación está formada por las siguientes tareas:
    - **AzureResourceManagerTemplateDeployment** usa **Bicep** para implementar una instancia de Azure Container Registry.
    - La tarea de **PowerShell** toma la salida de Bicep (servidor de inicio de sesión acr) y crea una variable de canalización.
    - La tarea **DockerCompose** compila e inserta las imágenes de contenedor para eShopOnWeb en Azure Container Registry.

1. La canalización tomará un nombre en función del nombre del proyecto. Permite **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones > Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar el nombre/Quitar**. Asígnale el nombre **eshoponweb-ci-dockercompose** y haz clic en **Guardar**.

1. Una vez finalizada la ejecución, en Azure Portal, abre el grupo de recursos definido previamente y verás la instancia de Azure Container Registry (ACR) con las imágenes de contenedor creadas, **eshoppublicapi** y **eshopwebmvc**. Solo usarás **eshopwebmvc** en la fase de implementación.

    ![Captura de pantalla de imágenes de contenedor en ACR.](images/azure-container-registry.png)

1. Haga clic en **Claves de acceso**, habilite el **Usuario administrador** si aún no lo ha hecho y copie el valor de **contraseña**. Se usará en la siguiente tarea, ya que lo mantendremos como secreto en Azure Key Vault.

    ![Captura de pantalla de la ubicación de la contraseña de ACR.](images/acr-password.png)

#### Tarea 2: crear una instancia del Almacén de claves de Azure

En esta tarea, crearás una instancia del Almacén de claves de Azure mediante el portal de Azure.

En este escenario de laboratorio, tendremos una instancia de Azure Container (ACI) que extrae y ejecuta una imagen de contenedor almacenada en Azure Container Registry (ACR). Tenemos previsto almacenar la contraseña de ACR como un secreto en el almacén de claves.

1. En el portal de Azure, en el cuadro de texto **Buscar recursos, servicios y documentos**, escribe **`Key vault`** y presiona la tecla **Entrar**.
1. En la hoja **Almacén de claves**, seleccione **Crear > Almacén de claves**.
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

En esta tarea, crearás un grupo de variables en Azure DevOps que recuperará el secreto de contraseña de ACR de Key Vault mediante la conexión de servicio creada previamente.

1. En el equipo de laboratorio, inicia un explorador web y navega hasta el proyecto de Azure DevOps **eShopOnWeb**.

1. En el panel de navegación vertical del portal de Azure DevOps, seleccione **Pipelines (Canalizaciones) > Library (Biblioteca)**. Haz clic en **+ Grupo de variables**.

1. En el panel **Nuevo grupo de variables**, configura las opciones siguientes:

    | Configuración | Valor |
    | --- | --- |
    | Nombre del grupo de variables | **eshopweb-vg** |
    | Vincular secretos desde un Almacén de claves de Azure | **enable** |
    | Suscripción de Azure | **Conexiones disponibles del servicio de Azure > Suscripciones de Azure** |
    | Nombre del almacén de claves | Nombre del almacén de claves|

1. En **Variables**, haz clic en **+ Agregar** y selecciona el secreto **acr-secret**. Haga clic en **Aceptar**.
1. Haga clic en **Guardar**.

    ![Captura de pantalla de la creación del grupo de variables.](images/vg-create.png)

#### Tarea 4: configurar la canalización de CD para implementar el contenedor en instancias de Azure Container (ACI)

En esta tarea, importarás una canalización de CD, la personalizarás y ejecutarás para implementar la imagen de contenedor creada antes en una instancia de contenedor de Azure.

1. En el equipo de laboratorio, abre un explorador web y ve al proyecto de Azure DevOps **eShopOnWeb**. Ve a **Canalizaciones > Canalizaciones** y haz clic en **Nueva canalización**.

1. En la ventana **¿Dónde está el código?**, selecciona **Azure Repos Git (YAML)** y selecciona el repositorio **eShopOnWeb**.

1. En la sección **Configurar**, selecciona **Archivo YAML de Azure Pipelines existente**. Seleccionar rama: **principal**, proporcione la siguiente ruta de acceso **/.ado/eshoponweb-cd-aci.yml** y haga clic en **Continuar**.

1. En la definición de canalización de YAML, personaliza los siguientes elementos:

    - Reemplaza **YOUR-SUBSCRIPTION-ID** con el identificador de la suscripción a Azure.
    - **az400eshop-NAME** reemplace NAME para que sea único globalmente.
    - ** YOUR-ACR.azurecr.io** y **ACR-USERNAME** con el servidor de inicio de sesión de ACR (ambos necesitan el nombre de ACR, se pueden revisar en ACR > Claves de acceso).
    - Reemplaza **AZ400-EWebShop-NAME** con el nombre del grupo de recursos definido antes en el laboratorio.

1. Haz clic en **Guardar y ejecutar**.
1. Abre la canalización y espera a que se ejecute correctamente.

    > **Importante**: si ves el mensaje “Esta canalización necesita permisos para acceder a los recursos antes de que la ejecución pueda pasar de Docker Compose a ACI”, haz clic en Ver, Permitir y Permitir nuevamente. Esto es necesario para permitir que la canalización cree el recurso.

    > **Nota**: la implementación puede tardar unos minutos en completarse. La definición de CD consta de las siguientes tareas:
    - **Recursos**: están preparados para desencadenarse automáticamente en función de la finalización de la canalización de CI. También descarga el repositorio para el archivo Bicep.
    - **Variables (para la fase de implementación)** se conecta con el grupo de variables para utilizar el secreto del Almacén de claves de Azure, **acr-secret**
    - **AzureResourceManagerTemplateDeployment** implementa la instancia de Azure Container (ACI) mediante la plantilla de bicep y proporciona los parámetros de inicio de sesión de ACR para permitir que ACI descargue la imagen de contenedor creada anteriormente desde Azure Container Registry (ACR).

1. La canalización tomará un nombre en función del nombre del proyecto. Permite **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones > Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar el nombre/Quitar**. Asígnale el nombre **eshoponweb-cd-aci** y haz clic en **Guardar**.

   > [!IMPORTANT]
   > Recuerda eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisar

En este laboratorio, integrarás el Almacén de claves de Azure con una canalización de Azure DevOps siguiendo estos pasos:

- Instancia de Azure Key Vault creada para almacenar una contraseña de ACR como secreto.
- Acceso facilitado a secretos en Azure Key Vault.
- Permisos configurados para leer el secreto.
- Canalización configurada para recuperar la contraseña de Azure Key Vault y pasarla a las tareas posteriores.
- Imagen de contenedor implementada en Azure Container Instance (ACI) mediante el secreto.
- Grupo de variables creado conectado a Azure Key Vault
