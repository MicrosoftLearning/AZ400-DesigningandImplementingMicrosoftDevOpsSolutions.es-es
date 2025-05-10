---
lab:
  title: Planificación ágil y administración de carteras con Azure Boards
  module: 'Module 01: Implement development for enterprise DevOps'
---

# Planificación ágil y administración de carteras con Azure Boards

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops).

## Introducción al laboratorio

En este laboratorio, conocerá las herramientas y procesos de planeación de Agile y administración de carteras que proporciona Azure Boards y cómo pueden ayudarle a planear y administrar el trabajo en todo su equipo, así como hacerle un seguimiento. Explorará el trabajo pendiente del producto, el trabajo pendiente de sprint y los paneles de tareas que pueden realizar un seguimiento del flujo de trabajo durante una iteración. También veremos las herramientas mejoradas en esta versión para escalar para equipos y organizaciones más grandes.

## Objetivos

Después de completar este laboratorio, podrá:

- Administrar equipos, áreas e iteraciones.
- Administrar elementos de trabajo.
- Administrar sprints y capacidad.
- Personalizar paneles Kanban.
- Definir paneles.
- Personalizar el proceso de equipo.

## Tiempo estimado: 60 minutos

## Instrucciones

### Ejercicio 0: (omitir si se ha realizado) Configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale el nombre **eShopOnWeb** a tu proyecto. Establece la visibilidad como **Privado**.
1. Haz clic en **Avanzado** y especifica **Scrum** como el **Proceso de elemento de trabajo**.
 Haga clic en **Crear**.

    ![Captura de pantalla del panel Crear nuevo proyecto.](images/create-project.png)

### Ejercicio 1: administración de un proyecto con la metodología ágil

En este ejercicio, usarás Azure Boards para hacer varias tareas comunes de planeamiento ágil y administración de carteras, como la administración de equipos, áreas, iteraciones, elementos de trabajo, sprints y capacidad, personalización de paneles Kanban, definición de paneles y personalización de procesos de equipos.

#### Tarea 1: administrar equipos, áreas e iteraciones

En esta tarea, crearás un nuevo equipo y configurarás su área e iteraciones.

Cada proyecto nuevo se configura con un equipo predeterminado, que coincide con el nombre del proyecto. Puedes crear más equipos. A cada equipo se le puede conceder acceso a un conjunto de herramientas y recursos de equipo del método ágil. La capacidad de crear varios equipos ofrece la flexibilidad de elegir el equilibrio adecuado entre autonomía y colaboración en toda la empresa.

1. En tu ordenador de laboratorio, inicia un explorador web y navega hasta el portal de Azure DevOps en `https://aex.dev.azure.com`.

   > **Nota**: si se te solicita, inicia sesión con la cuenta de Microsoft asociada a tu suscripción de Azure DevOps.

1. Abre el proyecto **eShopOnWeb** de tu organización de Azure DevOps.

    > **Nota**: también puedes acceder directamente a la página del proyecto al ir a la dirección URL <https://dev.azure.com/YOUR-AZURE-DEVOPS-ORGANIZATION/PROJECT-NAME>, donde el marcador de posición YOUR-AZURE-DEVOPS-ORGANIZATION, representa tu nombre de cuenta, y el marcador de posición PROJECT-NAME representa el nombre del proyecto.

1. Haz clic en el icono de engranaje con la etiqueta **Configuración del proyecto** situada en la esquina inferior izquierda de la página para abrir la página **Configuración del proyecto**.

    ![Captura de pantalla de la página de configuración de Azure DevOps.](images/m1/project_settings_v1.png) 

1. En la sección **General**, seleccione la pestaña **Teams**. Ya hay un equipo predeterminado en este proyecto, **eShopOnWeb Team**, pero creará uno nuevo para este laboratorio. Haz clic en **Nuevo equipo**.

    ![Captura de pantalla de la ventana de configuración del proyecto de Teams.](images/m1/new_team_v1.png)

1. En el panel **Crear un nuevo equipo**, en el cuadro de texto **Nombre del equipo**, escribe **`EShop-Web`**, deja otras opciones con sus valores predeterminados y haz clic en **Crear**.

    ![Captura de pantalla de la página de equipos de configuración del proyecto.](images/m1/eshopweb-team_v1.png)

1. En la lista de **Equipos**, selecciona el equipo recién creado para ver sus detalles.

    > **Nota**: de forma predeterminada, serás miembro del nuevo equipo. Puedes usar esta vista para administrar funciones como la pertenencia al equipo, las notificaciones y los paneles.

1. Haz clic en el vínculo **Iteraciones y rutas de área** en la parte superior de la página **EShop-Web** para empezar a definir la programación y el ámbito del equipo.

    ![Captura de pantalla de la opción Iteraciones y rutas de área".](images/m1/EShop-WEB-iterationsareas_v1.png)

1. En la parte superior del panel **Paneles**, selecciona la pestaña **Iteraciones** y haz clic en **+ Seleccionar iteraciones.**

    ![Captura de pantalla de la página de configuración de iteraciones.](images/m1/EShop-WEB-select_iteration_v1.png)

1. Seleccione **eShopOnWeb\Sprint 1** y haga clic en **Guardar y cerrar**. Ten en cuenta que este primer sprint aparecerá en la lista de iteraciones, pero las fechas aún no están establecidas.
1. Selecciona **Sprint 1** y haz clic en los **puntos suspensivos (...)**. En el menú contextual, selecciona **Editar**.

     ![Captura de pantalla de la configuración de edición de la pestaña de iteraciones.](images/m1/EShop-WEB-edit_iteration_v1.png)

    > **Nota**: especifica la fecha de inicio como el primer día laborable de la semana pasada y cuenta 3 semanas de trabajo completas para cada sprint. Por ejemplo, si el 6 de marzo es el primer día laborable del sprint, el periodo dura hasta el 24 de marzo. El sprint 2 comienza el 27 de marzo, que es tres semanas después del 6 de marzo.

1. Repite el paso anterior para agregar el **Sprint 2** y el **Sprint 3**. Podrías decir que estamos en la segunda semana del primer sprint.

    ![Captura de pantalla de la configuración de sprints en la pestaña de iteraciones.](images/m1/EShop-WEB-3sprints_v1.png)

1. Todavía en el panel **Configuración del proyecto / Boards / Configuración del equipo**, en la parte superior del panel, seleccione la pestaña **Áreas**. Encontrará un área generada automáticamente con el nombre que coincida con el nombre del equipo.

    ![Captura de pantalla de la página de configuración de las áreas.](images/m1/EShop-WEB-areas_v1.png)

1. Haz clic en los puntos suspensivos (...) junto a la entrada **área predeterminada** y, en la lista desplegable, selecciona **Incluir subáreas**.

    ![Captura de pantalla de la opción de incluir subárboles en la pestaña de áreas".](images/m1/EShop-WEB-sub_areas_v1.png)

    > **Nota**: la configuración predeterminada para todos los equipos excluye las rutas de subárea. Lo cambiaremos para incluir subáreas para que el equipo pueda ver todos los elementos de trabajo de todos los equipos. O bien, el equipo de administración también puede optar por no incluir subáreas, lo que oculta automáticamente los elementos de trabajo en cuanto se les asigna a uno de los equipos.

#### Tarea 2: administrar elementos de trabajo

En esta tarea, aprenderás las tareas comunes de administración de elementos de trabajo.

Los elementos de trabajo cumplen un rol destacado en Azure DevOps. Ya sea que se describe el trabajo que se realizará, los impedimentos para publicar una versión, las definiciones de prueba u otros elementos clave, los elementos de trabajo son el caballo de batalla de los proyectos actuales. En esta tarea, se centrará en el uso de varios elementos de trabajo para configurar el plan para ampliar el sitio eShopOnWeb, con una sección de entrenamiento del producto. Aunque puede ser abrumador crear una parte tan importante del contenido de una empresa, será muy fácil de administrar con Azure DevOps y con el proceso Scrum.

> **Nota**: esta tarea está diseñada para enseñar varias formas de crear diferentes tipos de elementos de trabajo, así como para demostrar la amplitud de las características disponibles en la plataforma. En consecuencia, estos pasos no deben considerarse instrucciones prescriptivas para la administración de proyectos. Las características son lo suficientemente flexibles como para adaptarse a las necesidades de tus procesos, así que explora y experimenta a medida que vas avanzando.

1. Haz clic en el nombre del proyecto en la esquina superior izquierda del portal de Azure DevOps para volver a la página principal del proyecto.

1. En el panel de navegación vertical del portal de Azure DevOps, selecciona el icono **Paneles** y, luego, **Elementos de trabajo**.

    > **Nota**: hay muchas maneras de crear elementos de trabajo en Azure DevOps, y descubriremos algunas de ellas. A veces es tan sencillo como activar una fuera de un panel.

1. En la ventana **Elementos de trabajo**, haz clic en **+ Nuevo elemento de trabajo > Epopeya**.

    ![Captura de pantalla de la creación de un elemento de trabajo.](images/m1/EShop-WEB-create_epic_v1.png)

1. En el cuadro de texto **Escribir título**, escribe **`Product training`**.
1. En la esquina superior izquierda, selecciona la entrada **Nadie seleccionado** y, en la lista desplegable, selecciona tu cuenta de usuario para asignar el nuevo elemento de trabajo. Si su nombre no aparece, comience a escribirlo y haga clic en **Buscar**.
1. Junto a la entrada **Área**, selecciona la entrada **eShopOnWeb** y, en la lista desplegable, selecciona **EShop-WEB**. Esto establecerá el **Área** como **eShopOnWeb\EShop-WEB**.
1. Junto a la entrada **Iteración**, selecciona la entrada **eShopOnWeb** y, en la lista desplegable, selecciona **Sprint 2**. Esto establecerá la **iteración** como **eShopOnWeb\Sprint 2**.
1. Para terminar de hacer cambios, haz clic en **Guardar**. **No cierres**.

    ![Captura de pantalla de los detalles del elemento de trabajo, incluyendo las opciones de título, área e iteración.](images/m1/EShop-WEB-epic_details_v1.png)

    > **Nota**: normalmente, uno completa con la mayor cantidad de información posible, pero esto es suficiente para este laboratorio.

    > **Nota**: el formulario de elementos de trabajo tiene todos los ajustes de elementos de trabajo pertinentes. Esto incluye detalles sobre a quién está asignado, su estado en distintos parámetros y toda la información asociada, así como el historial que describe cómo se ha controlado desde la creación. Una de las áreas claves en las que debes centrarte es el **trabajo relacionado**. Descubriremos una de las formas de agregar una característica a esta epopeya.

1. En la sección **Trabajo relacionado** en la parte inferior derecha, selecciona la entrada **Agregar vínculo** y, en la lista desplegable, selecciona **Nuevo elemento**.
1. En el panel **Agregar vínculo**, en la lista desplegable **Tipo de vínculo**, selecciona **Secundario**. A continuación, en la lista desplegable **Tipo de elemento de trabajo**, selecciona **Característica**, en el cuadro de texto **Título**, escribe **`Training dashboard`**.

    ![Captura de pantalla de la creación del vínculo de un elemento de trabajo.](images/m1/EShop-WEB-create_child_feature.png)

1. Haz clic en **Agregar vínculo** para guardar el elemento secundario.

    ![Captura de pantalla del área de trabajo relacionada con el elemento de trabajo.](images/m1/EShop-WEB-epic_with_linked_item_v1.png)

    > **Nota**: en el **panel de entrenamiento**, ten en cuenta que la asignación, el **área** y la **iteración** ya están establecidos en los mismos valores de la epopeya en la que se basa la característica. Asimismo, la característica se vincula automáticamente con el elemento primario desde el cual se creó.

1. En el **Panel de formación** (Nueva característica), haz clic en **Guardar y cerrar**.

1. En el panel de navegación vertical del portal de Azure DevOps, en la lista de elementos **Paneles**, selecciona **Paneles**.
1. En el panel **Paneles**, selecciona la entrada **paneles de EShop-WEB**. Se abrirá el panel para ese equipo determinado.

    ![Captura de pantalla de la selección del panel EShop-WEB.](images/m1/EShop-WEB-_boards_v1.png)

1. En el panel **Paneles**, en la esquina superior derecha, selecciona la entrada **Elementos de trabajos pendientes** y, en la lista desplegable, selecciona **Características**.

    > **Nota**: esto facilita la adición de tareas y otros elementos de trabajo a las características.

1. Mantén el puntero del mouse sobre el rectángulo que representa la característica **Panel de entrenamiento**. Aparecerán los puntos suspensivos (...) en la esquina superior derecha.
1. Haz clic en el icono de puntos suspensivos (...) y, en la lista desplegable, selecciona **Agregar elemento de trabajo pendiente**.

    ![Captura de pantalla de la opción Agregar elemento de trabajo pendiente en la característica Panel de entrenamiento.](images/m1/EShop-WEB-add_pb_v1.png)

1. En el cuadro de texto del nuevo elemento de trabajo pendiente, escribe **`As a customer, I want to view new tutorials`** y presiona la tecla **Entrar** para guardar la entrada.

    > **Nota**: se crea un nuevo elemento de trabajo de pendiente del producto (PBI, por sus siglas en inglés), que es un elemento secundario de la característica y comparte su área e iteración.

1. Repite el paso anterior para agregar dos PBI más, diseñados para que el cliente vea los últimos tutoriales vistos y solicite nuevos tutoriales denominados, respectivamente, **`As a customer, I want to see tutorials I recently viewed`** y **`As a customer, I want to request new tutorials`**.

    ![Captura de pantalla de agregar trabajo pendiente.](images/m1/EShop-WEB-pbis_v1.png)

1. En el panel **Paneles**, en la esquina superior derecha, selecciona la entrada **Características** y, en la lista desplegable, selecciona **Elementos de trabajo pendiente**.

     ![Captura de pantalla de ver elementos de trabajo pendiente.](images/m1/EShop-WEB-backlog_v1.png)

    > **Nota**: los elementos de trabajo pendiente tienen un estado que define en qué etapa del proceso se encuentran. Aunque puedes abrir y editar el elemento de trabajo con el formulario, es más fácil arrastrar las tarjetas hacia el panel.

1. En la pestaña **Panel** del panel **EShop-WEB**, arrastra el primer elemento de trabajo denominado **Como cliente, me gustaría ver nuevos tutoriales** de las fases **Nuevo** a **Aprobado**.

    ![Captura de pantalla del panel con los elementos de trabajo.](images/m1/EShop-WEB-new2ap_v1.png)

    > **Nota**: también puedes expandir tarjetas de elementos de trabajo para obtener detalles editables de manera práctica.

1. Mantén el puntero del mouse sobre el rectángulo que representa el elemento de trabajo que has movido a la fase **Aprobado**. Aparecerá el símbolo de intercalación hacia abajo.
1. Haga clic en el símbolo de intercalación hacia abajo para desplegar la tarjeta del elemento de trabajo, sustituya la entrada **Sin asignar** por su nombre y, a continuación, seleccione su cuenta para asignarse el PBI trasladado.
1. En la pestaña **Panel** del panel **EShop-WEB**, arrastra el segundo elemento de trabajo denominado **Como cliente, me gustaría ver los últimos tutoriales que he visto** desde la fase **Nuevo** hasta **Confirmado**.
1. En la pestaña **Panel** del panel **EShop-WEB**, arrastra el tercer elemento de trabajo denominado **Como cliente, me gustaría solicitar nuevos tutoriales** de la fase **Nuevo** hasta **Listo**.

    ![Captura de pantalla del panel con elementos de trabajo movidos a las columnas especificadas en los pasos anteriores.](images/m1/EShop-WEB-board_pbis_v1.png)

    > **Nota**: el panel de tareas es una vista en los trabajos pendientes. También puedes usar la vista tabular.

1. En la pestaña **Panel** del panel **EShop-WEB**, en la parte superior del panel, haz clic en **Ver como trabajo pendiente** para mostrar el formulario tabular.

    ![Captura de pantalla del trabajo pendiente de EShop-WEB.](images/m1/EShop-WEB-view_backlog_v1.png)

    > **Nota**: puedes usar el signo más situado justo debajo de la etiqueta de pestaña **Trabajo pendiente** del panel **EShop-WEB** para ver las tareas anidadas en estos elementos de trabajo.

    > **Nota**: puedes usar el segundo signo más situado justo a la izquierda del primer elemento de trabajo pendiente para agregarle una nueva tarea.

1. En la pestaña **Trabajo pendiente** del panel **EShop-WEB**, en la esquina superior izquierda del panel, haga clic en el signo más situado junto al primer elemento de trabajo. Aparecerá el panel **NUEVA TAREA**.

    ![Captura de pantalla de la opción crear tarea en la lista de trabajos pendientes.](images/m1/new_task_v1.png)

1. En la parte superior del panel **NUEVA TAREA**, en el cuadro de texto **Escribir título**, escribe **`Add page for most recent tutorials`**.
1. En el panel **NUEVA TAREA**, en el cuadro de texto **Trabajo restante**, escribe **5**.
1. En el panel **NUEVA TAREA**, en la lista desplegable **Actividad**, selecciona **Desarrollo**.
1. En el panel **NUEVA TAREA**, haz clic en **Guardar y cerrar**.

    ![Captura de pantalla de la creación de un elemento de trabajo nuevo.](images/m1/EShop-WEB-save_task_v1.png)

1. Repite los últimos cinco pasos para agregar otra tarea denominada **`Optimize data query for most recent tutorials`**. Configura tu **Trabajo restante** con el valor **3** y elige la **Actividad**: **Diseño**. Haz clic en **Guardar y cerrar** cuando termines.

#### Tarea 3: administrar sprints y capacidad

En esta tarea, aprenderás las tareas más frecuentes de un sprint y de administración de capacidad.

El equipo crea el trabajo pendiente de sprint durante la reunión de planificación de sprints, que suele realizarse el primer día del sprint. Cada sprint corresponde a un intervalo de tiempo que admite la capacidad del equipo para trabajar con herramientas y procesos de la metodología ágil. Durante la reunión de planificación, el propietario del producto trabaja con su equipo para identificar esos casos o elementos de trabajo pendiente que se van a completar en el sprint.

Las reuniones de planificación normalmente constan de dos partes. En la primera parte, el equipo y el propietario del producto identifican los elementos de trabajo pendiente que el equipo siente que puede comprometerse a completar en el sprint, en función de la experiencia con sprints anteriores. Estos elementos se agregan al trabajo pendiente de sprint. En la segunda parte, el equipo determina cómo desarrollará y verificará cada elemento. Después, definen y calculan las tareas necesarias para completar cada elemento. Por último, el equipo se compromete a implementar algunos o todos los elementos en función de estos cálculos.

El trabajo pendiente de sprint debe contener toda la información que el equipo necesita para completar correctamente el trabajo dentro del tiempo asignado sin tener que apresurarse a último momento. Antes de planear el sprint, te recomendamos haber creado, priorizado y calculado el trabajo pendiente, y haber definido los sprints.

1. En el panel de navegación vertical del portal de Azure DevOps, selecciona el icono **Paneles** y, en la lista de elementos **Paneles**, selecciona **Sprints**.
1. En la pestaña **Panel de tareas** de la vista **Sprints**, en la barra de herramientas a la derecha, selecciona el símbolo ** Ver opciones** (directamente a la izquierda del icono de embudo) y, en la lista desplegable **Ver opciones**, selecciona la entrada **Detalles del trabajo**. Seleccione **Sprint 2** como filtro.

    ![Captura de pantalla del panel EShop-WEB con los detalles del trabajo.](images/m1/EShop-WEB-work_details_v1.png)

    > **Nota**: el sprint actual tiene un ámbito bastante limitado. Hay dos tareas en la fase **Tareas pendientes**. En este momento, no se ha asignado ninguna tarea. Ambas muestran un valor numérico a la derecha de la entrada **Sin asignar** que representa el cálculo del trabajo restante.

1. En la columna **ToDo**, fíjese en el elemento de tarea **Agregar página de tutoriales más recientes**, haga clic en la entrada **Sin asignar** y, en la lista de cuentas de usuario, seleccione su cuenta para asignarse la tarea.

1. Selecciona la pestaña **Capacidad** de la vista **Sprints**.

    ![Captura de pantalla del panel con los elementos de trabajo con horas de capacidad.](images/m1/EShop-WEB-capacity_v1.png)

    > **Nota**: esta vista permite definir qué actividades puede realizar un usuario y en qué nivel de capacidad.

1. En la pestaña **Capacidad** de la vista **Sprints**, para tu cuenta de usuario, establece el campo **Actividad** en **Desarrollo** y, en el cuadro de texto **Capacidad por día**, escribe **1**. A continuación, haga clic en **Save**(Guardar).

    > **Nota**: esto representa 1 hora de trabajo de desarrollo al día. Ten en cuenta que puedes agregar actividades adicionales por usuario en caso de que hagan otras tareas además de desarrollo.

    ![Captura de pantalla de la configuración de capacidad.](images/m1/EShop-WEB-capacity-setdevelopment_v1.png)

    > **Nota**: Supongamos que también va a tomar algunas vacaciones. También se debe agregar a la vista de capacidad.

1. En la pestaña ** Capacidad** de la vista **Sprints**, situada junto a la entrada que representa la cuenta de usuario, en la columna **Días libres**, haz clic en la entrada **0 días**. Aparecerá un panel en el que podrás establecer tus días libres.
1. En el panel que se visualiza, usa la vista de calendario para establecer tus vacaciones de modo que abarquen cinco días laborables durante el sprint actual (dentro de las próximas tres semanas) y, cuando termines, haz clic en **Aceptar**.

    ![Captura de pantalla de la configuración del día libre.](images/m1/EShop-WEB-days_off_v1.png)

1. Cuando vuelvas a la pestaña **Capacidad** de la vista **Sprints**, haz clic en **Guardar**.
1. Selecciona la pestaña **Panel de tareas** de la vista **Sprints**.

    ![Captura de pantalla de la sección de detalles de trabajo.](images/m1/EShop-WEB-work_details_window_v1.png)

    > **Nota:** ten en cuenta que el panel **Detalles del trabajo** se ha actualizado para reflejar el ancho de banda disponible. El número real que se muestra en el panel **Detalles del trabajo** puede variar, pero la capacidad total del sprint será igual que la cantidad de días laborables restantes hasta el final del sprint, ya que asignó 1 hora al día. Toma nota de este valor, ya que lo usarás en los próximos pasos.

    > **Nota**: una característica práctica de los paneles es que puedes actualizar fácilmente los datos clave en línea. Se recomienda actualizar periódicamente el cálculo de **Trabajo restante** para reflejar el tiempo previsto para cada tarea. Supongamos que has revisado el trabajo de la tarea **Agregar página para los tutoriales más recientes** y has descubierto que tardará más de lo esperado.

1. En la pestaña **Panel de tareas** de la vista **Sprints**, en el cuadro que representa **Agregar página para los tutoriales más recientes**, establece la cantidad prevista de horas en **14**, para que coincida con la capacidad total de este sprint, que has identificado en el paso anterior.

    ![Captura de pantalla de la capacidad de tiempo asignada por el equipo y el panel.](images/m1/EShop-WEB-over_capacity_v1.png)

    > **Nota**: esto amplía automáticamente el **Desarrollo** y tus capacidades personales al máximo. Puesto que son lo suficientemente grandes como para cubrir las tareas asignadas, permanecen en verde. Sin embargo, la capacidad global del **Equipo** se supera debido a las 3 horas adicionales que requiere la tarea **Optimizar datos de consulta para los últimos tutoriales vistos**.

    > **Nota**: una manera de resolver este problema de capacidad sería mover la tarea a una iteración futura. Puedes hacerlo de distintas maneras. Por ejemplo, podrías abrir la tarea aquí y editarla en el panel que proporciona acceso a los detalles de la tarea. Otro enfoque sería usar la vista **Trabajo pendiente**, que proporciona una opción de menú en línea para moverla. De todos modos, aún no muevas la tarea.

1. En la pestaña **Panel de tareas** de la vista **Sprints**, en la barra de herramientas a la derecha, selecciona el símbolo **Ver opciones** (directamente a la izquierda del icono de embudo) y, en la lista desplegable **Ver opciones**, selecciona la entrada **Asignado a**.

    > **Nota**: esto ajusta la vista para que puedas revisar el progreso de las tareas por persona, en lugar de por elemento de trabajo pendiente.

    > **Nota**: también hay varias tareas de personalización disponibles.

1. Haz clic en el icono de engranaje **Configurar ajustes del equipo** (situado a la derecha del icono de embudo).
1. En el panel **Configuración**, selecciona la pestaña **Estilos**, haz clic en **+ Agregar regla de estilos**; en la etiqueta **Nombre de la regla**, en el cuadro de texto **Nombre**, escribe **`Development`** y, en la lista desplegable **Color**, selecciona el rectángulo verde.

    > **Note**: todas las tarjetas aparecerán de color verde si cumplen los criterios de regla establecidos justo debajo del nombre de regla.

1. En la sección bajo el nombre de regla, en la lista desplegable **Campo**, selecciona **Actividad**, en la lista desplegable **Operador**, selecciona **=** y, en la lista desplegable **Valor**, selecciona **Desarrollo**.

    ![Captura de pantalla de la configuración del estilo del panel.](images/m1/EShop-WEB-styles_v2.JPG)

1. Haz clic en **Guardar** para guardar y cerrar la configuración.

    ![Captura de pantalla de los estilos de tareas en el panel de equipos.](images/m1/EShop-WEB-sprint-green_v1.png)

    > **Nota**: todas las tarjetas asignadas a actividades de **Desarrollo** aparecerán de color verde.

1. Vuelve al panel **Configuración**, selecciona la pestaña **General** y, en la sección **Trabajos pendientes**, ve y configura los niveles de navegación.

    > **Nota**: las epopeyas no se incluyen de forma predeterminada, pero puedes cambiar esto.

1. En el panel **Configuración**, selecciona la pestaña **General** y, en la sección **Días laborables**, especifica los días laborables que sigue el equipo.

    > **Nota**: esto se aplica a los cálculos de capacidad y de evolución.

1. En el panel **Configuración**, selecciona la pestaña **General** y, en la sección **Trabajar con errores**, puedes especificar cómo administrar errores en trabajos pendientes y paneles.

    > **Nota**: las entradas de esta pestaña permiten especificar cómo se presentan los errores en el panel.

1. En el panel **Configuración**, haz clic en **Guardar** para guardar y cerrar la regla de estilo.

    > **Nota**: la tarea asociada al **Desarrollo** ahora aparece en verde y es muy fácil de identificar.

#### Tarea 4: personalizar paneles Kanban

En esta tarea, aprenderás a personalizar paneles Kanban.

Para maximizar la capacidad de un equipo de ofrecer software de alta calidad de forma constante, Kanban destaca dos procedimientos principales. El primero consiste en visualizar el flujo de trabajo, y requiere asignar las fases de flujo de trabajo del equipo y configurar un panel Kanban que coincida. El segundo consiste en restringir la cantidad de trabajo en curso. Esto requiere establecer los límites de trabajo en curso (WIP, por su sigla en inglés). Después, estará listo para realizar un seguimiento del progreso en el panel Kanban y supervisar las métricas clave para reducir el plazo o el tiempo de ciclo. El panel Kanban convierte el trabajo pendiente en un panel interactivo, de modo que se proporciona un flujo de trabajo visual. A medida que avanza el trabajo desde su concepción hasta la finalización, puede actualizar los elementos en el panel. Cada columna representa una fase de trabajo y cada tarjeta representa un caso de usuario (tarjetas azules) o un error (tarjetas rojas) en esa fase de trabajo. Sin embargo, cada equipo desarrolla su propio proceso a lo largo del tiempo, por lo que la capacidad de personalizar el panel Kanban para que coincida con la forma en que el equipo funciona es fundamental para la entrega correcta.

1. En el panel de navegación vertical del portal de Azure DevOps, en la lista de elementos **Paneles**, selecciona **Paneles**.
1. En el panel **Boards**, haga clic en el icono de engranaje **Configurar ajustes del panel** (directamente a la derecha del icono de embudo).

    > **Nota**: el equipo hace hincapié en el trabajo realizado con los datos, por lo que se presta especial atención a cualquier tarea asociada al acceso o el almacenamiento de datos.

1. En el panel **Configuración**, selecciona la pestaña **Colores de etiqueta**, haz clic en **+ Agregar color de etiqueta**, en el cuadro de texto **Etiqueta**, escribe **`data`** y selecciona el rectángulo amarillo.

    ![Captura de pantalla de la configuración de la etiqueta.](images/m1/EShop-WEB-tag_color_v1.png)

    > **Nota**: cada vez que se etiqueta un elemento de trabajo pendiente o un error con **datos**, esa etiqueta se resaltará.

1. Selecciona la pestaña **Anotaciones**.

    > **Nota**: puedes especificar qué **Anotaciones** quieres incluir en las tarjetas para que sean más fáciles de leer y navegar. Cuando se habilita una anotación, es fácil acceder a los elementos de trabajo secundarios de ese tipo haciendo clic en la visualización en cada tarjeta.

1. Seleccione la pestaña **Pruebas**.

    > **Nota**: la pestaña **Pruebas** te permite configurar cómo se ven y se comportan las pruebas en las tarjetas.

1. Haz clic en **Guardar** para guardar y cerrar la configuración.
1. Desde la pestaña **Panel** del panel **EShop-WEB**, abra el elemento de trabajo que representa el elemento de trabajo pendiente **Como cliente, quiero ver nuevos tutoriales**.
1. En la vista detallada de elementos, en la parte superior del panel, a la derecha de la entrada **0 comentarios**, haga clic en **Añadir etiqueta**.
1. En el cuadro de texto que aparece, escribe **`data`** y presiona la tecla **Entrar**.
1. Repite el paso anterior para agregar la etiqueta **`ux`**.
1. Haz clic en **Guardar y cerrar** para guardar estas modificaciones.

    ![Captura de pantalla de las dos nuevas etiquetas visibles en la tarjeta.](images/m1/EShop-WEB-tags_v1.png)

    > **Nota**: ahora las dos etiquetas están visibles en la tarjeta, con la etiqueta **datos** resaltada en amarillo.

1. En el panel **Boards**, haga clic en el icono de engranaje **Configurar ajustes del panel** (directamente a la derecha del icono de embudo).
1. En el panel **Configuración**, selecciona la pestaña **Columnas**.

    > **Nota**: esta sección permite agregar nuevas fases al flujo de trabajo.

1. Haz clic en **+ Agregar columna**, bajo la etiqueta **Nombre de columna**, en el cuadro de texto **Nombre**, escribe **`QA Approved`** y, en el cuadro de texto **Límite de WIP**, escribe **1**.

    > **Nota**: el límite de trabajo en curso de 1 indica que solo puede haber un elemento de trabajo a la vez en esta fase. Normalmente, establecerías un valor más alto, pero solo hay dos elementos de trabajo para demostrar la característica.

    ![Captura de pantalla de la configuración de WIP.](images/m1/EShop-WEB-qa_column_v1.png)

1. Fíjese en los puntos suspensivos junto a la columna **QA aprobado** que ha creado. Seleccione **Mover a la derecha** dos veces, para que la columna QA aprobado se sitúe entre **Confirmado** y **Hecho**.
1. Haz clic en **Guardar** para guardar y cerrar la configuración.

1. En el **portal de Boards**, la columna **QA aprobado** ahora está visible en la vista del panel Kanban.
1. Arrastra el elemento de trabajo **Como cliente, me gustaría ver los últimos tutoriales que he visto** desde la fase **Confirmado** hasta **Control de calidad aprobado**.
1. Arrastra el elemento de trabajo **Como cliente, me gustaría ver nuevos tutoriales** desde la fase **Aprobado** hasta **Control de calidad aprobado**.

    ![Captura de pantalla del panel que supera su límite de WIP.](images/m1/EShop-WEB-wip_limit_v1.png)

    > **Nota**: la fase ahora supera su límite **WIP** y aparece en rojo como advertencia.

1. Mueve el elemento de trabajo pendiente **Como cliente, me gustaría ver los últimos tutoriales que he visto** al estado **Confirmado**.
1. En el panel **Boards**, haga clic en el icono de engranaje **Configurar ajustes del panel** (directamente a la derecha del icono de embudo).
1. En el panel **Configuración**, vuelve a la pestaña **Columnas** y selecciona la pestaña **Control de calidad aprobado**.

    > **Nota**: suele haber una demora entre el momento en que el trabajo se mueve a una columna y el inicio real del trabajo. Para contrarrestar ese retraso y revelar el estado real del trabajo en curso, puede activar las columnas divididas. Cuando se dividen, cada columna tiene dos subcolumnas, **En curso** y **Listo**. Las columnas divididas permiten al equipo implementar un modelo de extracción. Sin la división de columnas, los equipos insertan el trabajo para indicar que han completado su fase de trabajo. Sin embargo, enviarlo a la siguiente fase no significa necesariamente que un miembro del equipo inicie inmediatamente el trabajo en ese elemento.

1. En la pestaña **Control de calidad aprobado**, habilita la casilla **Dividir la columna en En curso y Listo** para crear dos columnas independientes.

    > **Nota**: a medida que el equipo actualiza el estado del trabajo mientras avanza de una fase a la siguiente, esto ayuda a que lleguen a un acuerdo respecto a lo que significa **Listo**. Al especificar la **Definición de Listo** para cada columna Kanban, ayudas a compartir las tareas esenciales que se deben completar antes de mover un elemento a una fase de nivel inferior.

1. En la pestaña **Control de calidad aprobado**, en la parte inferior del panel, en el cuadro de texto **Definición de Listo**, escribe **`Passes **all** tests`**.
1. Haz clic en **Guardar** para guardar y cerrar la configuración.

    ![Captura de pantalla de la columna dividida de configuración y la definición de la configuración finalizada.](images/m1/dd_v1.png)

    > **Nota**: la fase **Control de calidad aprobado** ahora tiene las columnas **En curso** y **Listo** . También puedes hacer clic en el símbolo informativo (con la letra **i** en un círculo) junto al encabezado de columna para leer la **Definición de Listo**. Es posible que tenga que actualizar el explorador para ver los cambios.

    ![Captura de pantalla de las columnas divididas para la columna Control de calidad aprobado.](images/m1/EShop-WEB-qa_2columns_v1.png)

1. En el panel **Boards**, haga clic en el icono de engranaje **Configurar ajustes del panel** (directamente a la derecha del icono del embudo).

    > **Nota**: el panel Kanban admite la capacidad de visualizar el flujo de trabajo a medida que pasas de Nuevo a Listo. Cuando agregas **calles**, también puedes ver el estado del trabajo que admite diferentes clases de nivel de servicio. Puede crear un carril para representar cualquier otra dimensión que admita sus necesidades de seguimiento.

1. En el panel **Configuración**, selecciona la pestaña **Calles**.
1. En la pestaña **Calles**, haz clic en **+ Agregar calle**, debajo de la etiqueta **Nombre de la calle**, en el cuadro de texto **Nombre**, escribe **`Expedite`**.
1. En la lista desplegable **Color**, selecciona el rectángulo **Verde**.
1. Haz clic en **Guardar** para guardar y cerrar la configuración.

    ![Captura de pantalla de la creación acelerada de calles.](images/m1/EShop-WEB-swimlane_v1.png)

1. Cuando vuelvas a la pestaña **Panel** del tablero **Paneles**, arrastra y coloca el elemento de trabajo **Confirmado** en la fase **Control de calidad aprobado\|En curso** por la calle **Acelerar** para que se reconozca como prioritario cuando el ancho de banda de control de calidad esté disponible.

    > **Nota**: Es posible que tenga que actualizar el explorador para que el calleno sea visible.

En este ejercicio has aprendido a usar el panel Kanban para visualizar el flujo de trabajo de una manera fácil de entender y realizar un seguimiento del progreso. También has aprendido a configurar el panel para que sea compatible con el proceso de tu equipo y cómo establecer el límite de trabajo en curso (WIP) para asegurarte de que el equipo no se sobrecarga con el trabajo.

### Ejercicio 2: definición de paneles

#### Tarea 1: Crear, personalizar y compartir paneles

En esta tarea, aprenderás el proceso de creación de paneles y sus componentes principales.

Los paneles permiten a los equipos visualizar el estado y supervisar el progreso en todo el proyecto. Rápidamente puedes tomar decisiones informadas sin tener que explorar en profundidad otras partes del sitio del proyecto del equipo. La página Información general proporciona acceso a un panel de equipo predeterminado que puedes personalizar agregando, quitando o reorganizando los iconos. Cada icono corresponde a un widget que proporciona acceso a una o varias características o funciones.

1. En el panel de navegación vertical del portal de Azure DevOps, selecciona el icono **Información general** y, en la lista de elementos **Información general**, selecciona **Paneles**.
1. Selecciona la **información general** del **equipo eShopOnWeb** y revisa el panel existente.

    ![Captura de pantalla de la página de información general de paneles.](images/m1/EShop-WEB-dashboard_v1.png)

1. En el panel **Paneles**, en la esquina superior derecha, selecciona **+ Nuevo panel**.

1. En el panel **Crear un panel**, en el cuadro de texto **Nombre**, escribe **`Product training`**, en la lista desplegable **Equipo**, selecciona el equipo **EShop-WEB** y haz clic en **Crear**.

    ![Captura de pantalla del panel de creación de paneles.](images/m1/EShop-WEB-create_dash_v1.png)

1. En el panel de nuevo panel, haz clic en **Agregar un widget**.
1. En el panel **Agregar widget**, en el cuadro de texto **Buscar widgets**, escribe **`sprint`** para encontrar los widgets existentes que se centren en los sprints. En la lista de resultados, selecciona **Información general de sprint** y haz clic en **Agregar**.
1. En el rectángulo que representa el widget recién agregado, haz clic en el icono de engranaje **Configuración** y revisa el panel **Configuración**.

    > **Nota**: el nivel de personalización variará según el widget.

1. En el panel **Configuración**, haz clic en **Cerrar** sin realizar ningún cambio.
1. Cuando vuelvas al panel **Agregar widget**, en el cuadro de texto **Buscar**, escribe **`sprint`** nuevamente para buscar widgets existentes que se centren en sprints. En la lista de resultados, selecciona **Capacidad de sprint** y haz clic en **Agregar**.

    > **Nota**: si el widget muestra "Establecer capacidad para usar el widget de capacidad de sprint", puedes seleccionar el vínculo **Establecer capacidad** para establecer la capacidad. Establece la actividad en Desarrollo y la capacidad en 1. Haz clic en **Guardar** y vuelve al panel.
    
1. En la vista **Panel**, en la parte superior del panel, haz clic en **Edición finalizada**.

    ![Captura de pantalla del panel con dos nuevos widgets.](images/m1/EShop-WEB-finished_dashboard_v1.png)

    > **Nota**: ahora puedes revisar dos aspectos importantes del sprint actual en el panel personalizado.

    > **Nota**: otra manera de personalizar paneles es generando gráficos basados en consultas de elementos de trabajo, que puedes compartir en un panel.

1. En el panel de navegación vertical del portal de Azure DevOps, selecciona el icono **Paneles** y, en la lista de elementos **Paneles**, selecciona **Consultas**.
1. En el panel **Consultas**, haz clic en **+ Nueva consulta**.
1. En la pestaña **Editor** del panel **Consultas > Mis consultas**, en la lista desplegable **Valor** de la fila **Tipo de elemento de trabajo**, selecciona **Tarea**.
1. Haz clic en **Agregar nueva cláusula** y, en la columna **Campo**, selecciona **Ruta de acceso del área** y, en la correspondiente lista desplegable **Valor**, selecciona **eShopOnWeb\\EShop-WEB**.
1. Haga clic en **Save**(Guardar).

    ![Captura de pantalla de la nueva consulta en el editor de consultas".](images/m1/EShop-WEB-query_v1.png)

1. En el cuadro de texto **Escribir nombre**, escribe **`Web tasks`**, en la lista desplegable **Carpeta**, selecciona **Consultas compartidas** y haz clic en **Aceptar**.
1. En el panel **Consultas > Consultas compartidas**, haz clic en **Tareas web** para abrir la consulta.
1. Selecciona la pestaña **Gráficos** y haz clic en **Nuevo gráfico**.
1. En el panel **Configurar gráfico**, en el cuadro de texto **Nombre**, escribe **`Web tasks - By assignment`**, en la lista desplegable **Agrupar por**, selecciona **Asignado a**, y haz clic en **Guardar gráfico** para guardar los cambios.

    ![Captura de pantalla del nuevo gráfico circular de tareas web.](images/m1/EShop-WEB-chart_v1.png)

    > **Nota**: ahora puedes agregar este gráfico a un panel.

1. Vuelva a la sección **Paneles** en el menú de **Información general**. En la sección **Eshop-Web**, seleccione el panel **Formación sobre productos** que usó anteriormente, para abrirlo.

1. Haz clic en **Editar** en el menú superior. En la lista **Agregar widget**, busca **`Chart`** , y selecciona **Gráfico para elementos de trabajo**. Haga clic en **Agregar** para agregar este widget al panel de EShop-Web.

1. Haga clic en el **configurar** (engranaje) dentro del **Gráfico para elementos de trabajo** para abrir la configuración del widget.

1. Acepte el título tal como está. En **Consulta**, seleccione **Consultas compartidas o tareas web**. Mantenga **Circular** para el tipo de gráfico. En **Agrupar por**, seleccione **Asignado a**. Mantenga los valores predeterminados de Agregación (recuento) y Ordenación (valor / ascendente).

1. Para confirmar la configuración, haga clic en **Guardar**.

1. Observe que el gráfico circular de resultados de la consulta se muestra en el panel. **Guarde** los cambios pulsando el botón **Edición finalizada** de la parte superior.

## Revisar

En este laboratorio has usado Azure Boards para realizar una serie de tareas comunes de planeamiento ágil y de administración de carteras, incluida la administración de equipos, áreas, iteraciones, elementos de trabajo, sprints y capacidad, personalización de paneles Kanban y definición de paneles.
