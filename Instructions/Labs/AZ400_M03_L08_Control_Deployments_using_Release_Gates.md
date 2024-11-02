---
lab:
  title: Control de implementaciones mediante puertas de versión
  module: 'Module 03: Design and implement a release strategy'
---

# Control de implementaciones mediante puertas de versión

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identifique una suscripción de Azure existente o cree una.

- Compruebe que tiene una cuenta Microsoft o una cuenta de Microsoft Entra con el rol Propietario en la suscripción de Azure y el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción de Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Introducción al laboratorio

En este laboratorio se describe la configuración de las puertas de implementación y se detalla cómo usarlas para controlar la ejecución de Azure Pipelines. Para ilustrar su implementación, configurará una definición de versión con dos entornos para una aplicación web de Azure. Realizarás la implementación en el entorno de DevTest solo cuando no haya errores de bloqueo para la aplicación y marcarás el entorno de DevTest como completado solo cuando no haya alertas activas en Application Insights de Azure Monitor.

Una canalización de versión especifica el proceso de versión completo para que una aplicación se implemente en varios entornos. Las implementaciones en cada fase se pueden automatizar completamente mediante trabajos y tareas. Lo ideal es no pretender que las nuevas actualizaciones de las aplicaciones se expongan simultáneamente a todos los usuarios. Se recomienda exponer actualizaciones por fases, es decir, exponerlas a un subconjunto de usuarios, supervisar su uso y exponerlas a otros usuarios en función de la experiencia del conjunto inicial de usuarios.

Las aprobaciones y puertas permiten ejercer el control del inicio y la finalización de las implementaciones en una versión. Puede esperar a que los usuarios aprueben o rechacen las implementaciones con aprobaciones manualmente. Con las puertas de versión, puede especificar los criterios de mantenimiento de la aplicación que se deben cumplir antes de promover la versión al siguiente entorno. Antes o después de cualquier implementación del entorno, todas las puertas especificadas se evalúan automáticamente hasta que pasan o alcanzan el período de tiempo de espera definido y producen un error.

Las puertas se pueden agregar a un entorno en la definición de versión desde las condiciones previas a la implementación o el panel de condiciones posteriores a la implementación. Se pueden agregar varias puertas a las condiciones del entorno para asegurarse de que todas las entradas son correctas para la versión.

Por ejemplo:

- Las puertas previas a la implementación garantizan que no haya ningún problema activo en el elemento de trabajo ni en el sistema de administración de problemas antes de implementar una compilación en un entorno.
- Las puertas posteriores a la implementación garantizan que no haya incidentes del sistema de supervisión o administración de incidentes de la aplicación después de implementarse y antes de promover la versión al siguiente entorno.

Hay cuatro tipos de puertas incluidas de forma predeterminada en cada cuenta.

- Invocar función de Azure: desencadene la ejecución de una función de Azure y asegúrese de que se complete correctamente.
- Consultar las alertas de Azure Monitor: observe las reglas de alertas de Azure Monitor configuradas para las alertas activas.
- Invocar API de REST: llame a una API de REST y continúe si devuelve una respuesta correcta.
- Consultar elementos de trabajo: asegúrese de que el número de elementos de trabajo correspondientes devueltos por una consulta esté dentro de un umbral.

## Objetivos

Después de completar este laboratorio, podrá:

- Configurar las canalizaciones de versión
- Configurar las puertas de versión
- Probar las puertas de versión

## Tiempo estimado: 75 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio.

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y deja los demás campos con los valores predeterminados. Haga clic en **Crear**.

   ![Captura de pantalla del panel Crear nuevo proyecto.](images/create-project.png)

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos > Archivos**, **Importar un repositorio**. Seleccione **importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

   ![Captura de pantalla del panel Importar repositorio.](images/import-repo.png)

1. El repositorio se organiza de la siguiente manera:
   - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
   - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
   - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
   - Definiciones de flujo de trabajo de GitHub del contenedor de carpetas **.github**.
   - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

1. Ve a **Repos > Ramas**.
1. Mantén el puntero sobre la rama **main** y haz clic en los puntos suspensivos a la derecha de la columna.
1. Haz clic en **Establecer como rama predeterminada**.

#### Tarea 3: Configurar canalización de CI como código con YAML en Azure DevOps

En esta tarea, agregarás una definición de compilación de YAML al proyecto existente.

1. Vuelve al panel **Canalizaciones** del centro de **Canalizaciones**.
1. En la ventana **Crear la primera canalización**, haz clic en **Crear canalización**.

   > **Nota**: usaremos el asistente para crear una nueva definición de canalización de YAML basada en nuestro proyecto.

1. En el panel **¿Dónde está el código?**, haz clic en la opción **Azure Repos Git (YAML)**.
1. En el panel **Seleccionar un repositorio**, haz clic en **eShopOnWeb**.
1. En el panel **Configurar tu canalización**, desplázate hacia abajo y selecciona **Archivo YAML de Azure Pipelines existente**.
1. En la hoja **Seleccionar un archivo YAML existente**, especifica los parámetros siguientes:
   - Rama: **principal**.
   - Ruta de acceso: **.ado/eshoponweb-ci.yml**
1. Haz clic en **Continuar** para guardar esta configuración.
1. En la pantalla **Revisar YAML de canalización**, haz clic en **Ejecutar** para iniciar el proceso de canalización de compilación.
1. Espera a que la canalización de compilación se complete correctamente. Omite las advertencias relacionadas con el propio código fuente, ya que no son pertinentes para este ejercicio de laboratorio.

   > **Nota**: Cada tarea del archivo YAML está disponible para su revisión, incluidas las advertencias y errores.

1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones > Canalizaciones** y haz clic en la canalización creada recientemente. Haz clic en los puntos suspensivos y en la opción **Cambiar de nombre o mover**. Asígnale el nombre **`eshoponweb-ci`** y haz clic en **Guardar**.

### Ejercicio 1: Crear los recursos de Azure necesarios para la canalización de versión

#### Tarea 1: crear dos aplicaciones web de Azure

En esta tarea, crearás dos aplicaciones web de Azure que representan los entornos **DevTest** y **Production**, en los que implementarás la aplicación a través de Azure Pipelines.

1. En el equipo del laboratorio, abre un explorador web, ve al [**Portal de Azure**](https://portal.azure.com) e inicia sesión con las credenciales de una cuenta de usuario con el rol Propietario en la suscripción que vas a usar en este laboratorio, así como el rol Administrador global en el inquilino de Microsoft Entra asociado a la suscripción.
1. Haz clic en el icono de **Cloud Shell** a la derecha del cuadro de texto de búsqueda en la parte superior de la página en el portal de Azure.
1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.

   > **Nota**: si es la primera vez que inicias **Cloud Shell** y aparece el mensaje **No tienes ningún almacenamiento montado**, selecciona la suscripción que utilizas en este laboratorio y haz clic en **Crear almacenamiento**.

1. En el símbolo del sistema **Bash**, en el panel de **Cloud Shell**, ejecuta el siguiente comando para crear un grupo de recursos (reemplaza el marcador de posición variable de `<region>` por el nombre de la región de Azure que hospedará las dos aplicaciones web de Azure, como "westeurope" o "centralus" o cualquier otra región disponible):

   > **Nota**: las posibles ubicaciones se pueden encontrar ejecutando el siguiente comando. Usa el **Nombre** en `<region>`: `az account list-locations -o table`

   ```bash
   REGION='centralus'
   RESOURCEGROUPNAME='az400m03l08-RG'
   az group create -n $RESOURCEGROUPNAME -l $REGION
   ```

1. Crear un plan de App Service

   ```bash
   SERVICEPLANNAME='az400m03l08-sp1'
   az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1
   ```

1. Crea dos aplicaciones web con nombres de aplicación únicos.

   ```bash
   SUFFIX=$RANDOM$RANDOM
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-DevTest
   az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n RGATES$SUFFIX-Prod
   ```

   > **Nota**: registra el nombre de la aplicación web DevTest. Lo necesitará más adelante en este laboratorio.

1. Espera a que el proceso de aprovisionamiento de recursos de Web App Services se complete y cierra el panel de **Cloud Shell**.

#### Tarea 2: crear un recurso de Application Insights

1. En Azure Portal, usa el cuadro de texto **Buscar recursos, servicios y documentos** en la parte superior de la página para buscar **Application Insights**. Después, en la lista de resultados, selecciona **Application Insights**.
1. En la hoja **Application Insights** seleccione **+ Crear**.
1. En la hoja **Application Insights**, en la pestaña **Aspectos básicos**, especifica las siguientes opciones de configuración (deja las demás con los valores predeterminados):

   | Configuración        | Value                                                                                 |
   | -------------- | ------------------------------------------------------------------------------------- |
   | Resource group | **az400m03l08-RG**                                                                    |
   | Nombre           | el nombre de la aplicación web DevTest que has registrado en la tarea anterior                     |
   | Region         | la misma región de Azure donde implementamos las aplicaciones web en la tarea anterior |

1. Haga clic en **Revisar y crear** y, a continuación, en **Crear**.
1. Espere a que se complete el proceso de aprovisionamiento.
1. En Azure Portal, ve al grupo de recursos **az400m03l08-RG** que has creado en la tarea anterior.
1. En la lista de recursos, haz clic en la aplicación web **DevTest** que acabas de crear.
1. En la página de la aplicación web **DevTest**, en el menú vertical de la izquierda, en la sección **Supervisión**, haz clic en **Application Insights**.
1. En la hoja **Application Insights**, selecciona **Activar Application Insights**.
1. En la sección **Cambiar su recurso**, haz clic en la opción **Seleccionar recurso existente**, en la lista de recursos existentes, selecciona el recurso de Application Insight recién creado, haz clic en **Aplicar** y cuando se te pida confirmación, haz clic en **Sí**.
1. Espera hasta que el cambio surta efecto.

   > **Nota**: aquí crearás alertas de supervisión, que usarás en la parte posterior de este laboratorio.

1. En la misma opción de menú **Configuración** / **Application Insights** dentro de la aplicación web, selecciona **Ver datos de Application Insights**. Esto te redirige a la hoja de Application Insights en Azure Portal.
1. En la hoja de recursos de Application Insights, en la sección **Supervisión**, haz clic en **Alertas** y luego haz clic en **Crear > Regla de alerta**.
1. En la hoja **Crear una regla de alertas**, en la sección **Condición**, haz clic en el vínculo **Ver todas las señales** y escribe **Solicitudes**. En la lista de resultados, selecciona **Solicitudes con error**.
1. En la hoja **Crear una regla de alertas**, en la sección **Condición**, deja el **Umbral** establecido en ** Estático** y valida la otra configuración predeterminada de la siguiente manera:

   - Tipo de agregación: Count
   - Operador: Mayor que
   - Unidad: Recuento

1. En el cuadro de texto **Valor de umbral**, escribe **0** y haz clic en **Siguiente:Acciones**. No realices ningún cambio en la hoja **Acciones** y define los parámetros siguientes en la sección **Detalles**:

   | Configuración                                        | Value                            |
   | ---------------------------------------------- | -------------------------------- |
   | Gravedad                                       | **2: Advertencia**                   |
   | Nombre de la regla de alertas                                | **RGATESDevTest_FailedRequests** |
   | Opciones avanzadas: resolución automática de alertas | **desactivado**                      |

   > **Nota**: las reglas de alertas de métricas pueden tardar hasta 10 minutos en activarse.

   > **Nota**: puedes crear varias reglas de alertas en diferentes métricas, como disponibilidad < 99 por ciento, tiempo de respuesta del servidor > 5 segundos o excepciones de servidor > 0

1. Para confirmar la creación de la regla de alertas, haz clic en **Revisar y crear** y confirma una vez más haciendo clic en **Crear**. Espera a que la regla de alerta se cree correctamente.

### Ejercicio 2: Configurar la canalización de versión

En este ejercicio, configurarás una canalización de versión.

#### Tarea 1: Configurar tareas de versión

En esta tarea, configurarás las tareas de versión como parte de la canalización de versión.

1. En el proyecto **eShopOnWeb** del portal de Azure DevOps, en el panel de navegación vertical, selecciona **Canalizaciones** y luego en la sección **Canalizaciones**, haz clic en **Versiones**.
1. Haz clic en **Nueva canalización**.
1. En la ventana **Seleccionar una plantilla**, **elige** **Implementación de Azure App Service** (Implementa la aplicación en Azure App Service. Elige entre Aplicación web en Windows, Linux, contenedores, Function Apps o WebJobs en la lista **Destacados** de plantillas.
1. Haga clic en **Aplicar**.
1. En la ventana **Fase** que aparece, actualiza el nombre de fase predeterminado "Fase 1" a **DevTest**. Cierra la ventana emergente con el botón **X**. Ahora estarás en el editor gráfico de la Canalización de versión, donde se muestra la fase DevTest.
1. En la parte superior de la página, cambia el nombre de la canalización actual de **Nueva canalización de versión** a **eshoponweb-cd**.
1. Mantén el cursor sobre la fase DevTest y haz clic en el botón **Clonar** para copiar la fase DevTest en una fase adicional. Ponle a esta fase el nombre de **Producción**.

   > **Nota**: La canalización tiene ahora dos fases llamadas **DevTest** y **Producción**.

1. En la pestaña **Canalización**, selecciona el rectángulo **Agregar un artefacto** y selecciona **eshoponweb-ci** en el campo **Origen (canalización de compilación)**. Haz clic en **Agregar** para confirmar la selección del artefacto.
1. En el rectángulo **Artefactos**, observa el **Desencadenador de implementación continua** (rayo). Haz clic para abrir los ajustes del **Desencadenador de implementación continua**. Haz clic en **Deshabilitado** para alternar el modificador y habilitarlo. Deja el resto de opciones de configuración en el valor predeterminado y cierra el panel **Desencadenador de implementación continua** haciendo clic en la marca **x** de la esquina superior derecha.
1. En la fase **Entornos de DevTest**, haz clic en la etiqueta **1 trabajo, 1 tarea** y revisa las tareas de esta fase.

   > **Nota**: El entorno de DevTest tiene una tarea que, respectivamente, publica el paquete de artefactos en Azure Web App.

1. En el panel **Todas las canalizaciones > eshoponweb-cd**, asegúrate de que está seleccionada la fase **DevTest**. En la lista desplegable **Suscripción de Azure**, selecciona tu suscripción de Azure y haz clic en **Autorizar**. Si se te pide, autentícate mediante la cuenta de usuario con el rol de propietario en la suscripción de Azure.
1. Confirma que el tipo de aplicación está establecido en "Aplicación web en Windows". Luego, en la lista desplegable **Nombre del Servicio de aplicaciones**, selecciona el nombre de la aplicación web **DevTest**.
1. Selecciona la tarea **Implementar Azure App Service**. En el campo **Paquete o carpeta**, actualiza el valor predeterminado de "$(System.DefaultWorkingDirectory)/\*\*/\*.zip" a "$(System.DefaultWorkingDirectory)/\*\*/Web.zip".

   > **Nota**: verás un signo de exclamación junto a la pestaña Tareas porque hay que configurar las opciones de la fase de producción.

1. Abra el panel **Ajustes de aplicación y configuración** y escriba `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` en el cuadro **Configuración de la aplicación**.

1. En el panel **Todas las canalizaciones > eshoponweb-cd**, ve a la pestaña **Canalización** y, esta vez, en la fase **Producción**, haz clic en la etiqueta **1 trabajo, 1 tarea**. De forma similar a la fase DevTest anterior, completa la configuración de la canalización. En la pestaña Tareas/proceso Implementación de producción, en la lista desplegable **Suscripción de Azure**, selecciona la suscripción de Azure que usaste para la fase **Entorno de DevTest**, que se muestra en **Conexiones disponibles del servicio de Azure**, puesto que ya creamos la conexión de servicio antes, cuando autorizamos el uso de la suscripción.
1. En la lista desplegable **Nombre del Servicio de aplicaciones**, selecciona el nombre de la aplicación web **Prod**.
1. Selecciona la tarea **Implementar Azure App Service**. En el campo **Paquete o carpeta**, actualiza el valor predeterminado de "$(System.DefaultWorkingDirectory)/\*\*/\*.zip" a "$(System.DefaultWorkingDirectory)/\*\*/Web.zip".
1. Abra el panel **Ajustes de aplicación y configuración** y escriba `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development` en el cuadro **Configuración de la aplicación**.
1. En el panel **Todas las canalizaciones > eshoponweb-cd**, haz clic en **Guardar** y, en el cuadro de diálogo **Guardar**, haz clic en **Aceptar**.

   Ya has configurado correctamente la canalización de versión.

1. En la ventana del explorador que muestra el proyecto **eShopOnWeb**, en el panel de navegación vertical, en la sección **Canalizaciones**, haz clic en **Canalizaciones**.
1. En el panel **Canalizaciones**, haz clic en la entrada que representa la canalización de compilación **eshoponweb-ci** y luego haz clic en **Ejecutar canalización**.
1. En el panel **Ejecutar canalización**, acepta la configuración predeterminada y haz clic en **Ejecutar** para crear la canalización. **Espera a que finalice la compilación de la canalización**.

   > **Nota**: Después de que la compilación se realice correctamente, la versión se activará automáticamente y la aplicación se implementará en ambos entornos. Una vez completada correctamente la compilación de la canalización, valida las acciones de versión.

1. En el panel de navegación vertical, en la sección **Canalizaciones**, haz clic en **Versiones** y, en el panel **eshoponweb-cd**, haz clic en la entrada que representa la versión más reciente.
1. En la hoja **eshoponweb-cd > Release-1**, realiza un seguimiento del progreso de la versión y comprueba que la implementación en ambas aplicaciones web se haya completado correctamente.
1. Cambia a la interfaz de Azure Portal, ve al grupo de recursos **az400m03l08-RG** y en la lista de recursos, haz clic en la aplicación web **DevTest**. En la hoja de la aplicación web, haz clic en **Examinar** y comprueba que la página web (sitio web de comercio electrónico) se cargue correctamente en una nueva pestaña del explorador web.
1. Vuelve a la interfaz de Azure Portal. Esta vez, ve al grupo de recursos **az400m03l08-RG** y, en la lista de recursos, haz clic en la aplicación web **Producción**. En la hoja de la aplicación web, haz clic en **Examinar** y comprueba que la página web se cargue correctamente en una nueva pestaña del explorador web.
1. Cierra la pestaña del explorador web que muestra el sitio web **EShopOnWeb**.

   > **Nota**: Ya tienes la aplicación con CI/CD configurada. En el ejercicio siguiente, configuraremos validaciones de calidad como parte de una canalización de versión más avanzada.

### Ejercicio 3: Configurar validaciones de versión

En este ejercicio, configurarás validaciones de calidad en la canalización de versión.

#### Tarea 1: Configurar validaciones previas a la implementación para aprobaciones

En esta tarea, configurarás validaciones previas a la implementación.

1. Ve a la ventana del explorador web donde se muestra el portal de Azure DevOps y abre el proyecto **eShopOnWeb**. En el panel de navegación vertical, en la sección **Canalizaciones**, haz clic en **Versiones** y, en el panel **eshoponweb-cd**, haz clic en **Editar**.
1. En el panel **Todas las canalizaciones > eshoponweb-cd**, en el borde izquierdo del rectángulo que representa la fase **entorno de DevTest**, haz clic en la forma ovalada que representa las **condiciones anteriores a la implementación**.
1. En el panel **Condiciones previas a la implementación**, coloca el control deslizante **Aprobaciones previas a la implementación** en **Habilitado** y, en el cuadro de texto **Aprobadores**, escribe y selecciona el nombre de la cuenta de Azure DevOps.

   > **Nota**: en un escenario real, debe ser un alias de nombre de equipo de DevOps, no tu nombre.

1. **Guarda** la configuración de aprobación previa y cierra la ventana emergente.
1. Haz clic en **Crear versión** y confirma presionando el botón **Crear** desde la ventana emergente.
1. Verás el mensaje de confirmación verde que indica que se ha creado la "Versión-2". Haz clic en el vínculo "Versión-2" para ver sus detalles.
1. Verás que la fase **DevTest** está en estado **Pendiente de aprobación**. Haz clic en el botón **Aprobar**. Esto vuelve a activar DevTest Stage.

#### Tarea 2: Configurar validaciones posteriores a la implementación para Azure Monitor

En esta tarea, habilitarás la validación posterior a la implementación del entorno de DevTest.

1. Cuando vuelvas al panel **Todas las canalizaciones > eshoponweb-cd**, en el borde derecho del rectángulo que representa la fase **DevTest Environment**, haz clic en la forma ovalada que representa las **condiciones posteriores a la implementación**.
1. En el panel **Condiciones posteriores a la implementación**, coloca el control deslizante **Validaciones** en **Habilitado**, haz clic en **+ Agregar** y, en el menú emergente, haz clic en **Consultar alertas de Azure Monitor**.
1. En el panel **Condiciones posteriores a la implementación**, en la sección **Consultar alertas de Azure Monitor**, en la lista desplegable **Suscripción a Azure**, selecciona la entrada **conexión de servicio** que representa la conexión a tu suscripción a Azure y, en la lista desplegable **Grupo de recursos**, selecciona la entrada **az400m03l08-RG**.
1. En el panel **Condiciones posteriores a la implementación**, expande la sección **Opciones avanzadas** y configura las siguientes opciones:

   - Tipo de filtro: **Ninguno**
   - Gravedad: **Sev0, Sev1, Sev2, Sev3, Sev4**
   - Intervalo de tiempo: **Última hora**
   - Estado de alerta: **Confirmada, nueva**
   - Condición de supervisión: **Desencadenados**

1. En el panel **Condiciones posteriores a la implementación**, expande **Opciones de evaluación** y configura estas opciones:

   - Establece el valor de **Tiempo entre la reevaluación de las validaciones** en **5 minutos**.
   - Establece el valor de **Tiempo de espera tras el cual se produce un error de las validaciones** en **8 minutos**.
   - Selecciona la opción **Si las validaciones son correctas, pedir aprobaciones**.

   > **Nota**: El intervalo de muestreo y el tiempo de espera funcionan juntos para que las validaciones llamen a sus funciones a intervalos adecuados y rechacen la implementación si no se realizan correctamente durante el mismo intervalo de muestreo dentro del tiempo de espera.

1. Cierra el panel **Condiciones posteriores a la implementación** haciendo clic en la marca **x** en la esquina superior derecha.
1. Cuando vuelvas al panel **eshoponweb-cd**, haz clic en **Guardar** y, en el cuadro de diálogo **Guardar**, haz clic en **Aceptar**.

### Ejercicio 4: Probar validaciones de versión

En este ejercicio, probarás las validaciones de versión actualizando la aplicación, que desencadenará una implementación.

#### Tarea 1: Actualizar e implementar la aplicación después de agregar validaciones de versión

En esta tarea, primero generarás algunas alertas para la aplicación web DevTest y después harás un seguimiento del proceso de versión con las validaciones de versión habilitadas.

1. En Azure Portal, ve al recurso **DevTest Web App** implementado anteriormente.
1. En el panel Información general, verás el campo **URL** que muestra el hipervínculo de la aplicación web. Haga clic en este vínculo, que le redirige a la aplicación web eShopOnWeb en el explorador.
1. Para simular una **solicitud con error**, agrega **/discount** a la dirección URL, lo que dará como resultado un mensaje de error, ya que esa página no existe. Actualiza esta página varias veces para generar varios eventos.
1. En Azure Portal, en el campo "Buscar recursos, servicios y documentos", escribe **`Application Insights`** y selecciona el recurso **DevTest-AppInsights** creado en el ejercicio anterior. Ahora ve a **Alertas**.
1. Debe haber al menos **1** nueva alerta en la lista de resultados, con un nivel de **Gravedad 2**. Escribe **`Alerts`** para abrir el servicio de alertas de Azure Monitor.
1. Ten en cuenta que debe haber al menos **1** alerta con error marcada con **Gravedad 2: advertencia** en la lista. Esto se desencadena al validar la dirección URL no existente del sitio web en el ejercicio anterior.

   > **Nota:** Si aún no aparece ninguna alerta, espera unos minutos más.

1. Vuelva al portal de Azure DevOps y abra el proyecto **eShopOnWeb**. Ve a **Canalizaciones**, selecciona **Versiones** y luego, **eshoponweb-cd**.
1. Haz clic en el botón **Crear una versión**.
1. Espera a que se inicie la canalización de versión y **aprueba** la acción de versión de DevTest Stage.
1. Espera a que la fase de lanzamiento de DevTest se complete correctamente. Observa cómo la opción **Validaciones posteriores a la implementación** cambia a un estado **Validaciones de evaluación**. Haz clic en el icono **Validaciones de evaluación**.
1. Para **Consultar las alertas de Azure Monitor**, observa un estado de error inicial.
1. Deja la canalización de versión en un estado pendiente durante los próximos 5 minutos. Una vez transcurridos los 5 minutos, observa que la segunda evaluación de nuevo presenta un error.
1. Este es el comportamiento previsto, ya que hay una aplicación alerta de Application Insights desencadenada para la aplicación web DevTest.

   > **Nota**: Dado que hay una alerta desencadenada por la excepción, se producirá un error en la validación de **Consultar Azure Monitor**. Esto, a su vez, impedirá la implementación en el entorno de **producción**.

1. Espera un par de minutos y valida el estado de las validaciones de versión de nuevo. Pocos minutos después de comprobar las validaciones de versión iniciales y, dado que la alerta inicial de Application Insights se desencadenó con la acción "Desencadenados", debería dar lugar a una validación de versión correcta, lo que permite la implementación de la fase de lanzamiento de producción.

   > **Nota:** Si se produce un error en la validación, cierra la alerta.

   > [!IMPORTANT]
   > Recuerda eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisar

En este laboratorio, configuraste canalizaciones de versión y, luego, configuraste y probaste las validaciones de versión.
