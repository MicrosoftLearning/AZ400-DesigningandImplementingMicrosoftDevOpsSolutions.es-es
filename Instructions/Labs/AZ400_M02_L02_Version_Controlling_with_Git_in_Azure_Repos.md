---
lab:
  title: Control de versiones con Git en Azure Repos
  module: 'Module 02: Work with Azure Repos and GitHub'
---

# Control de versiones con Git en Azure Repos

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Si no tienes instalado Git 2.29.2 o una versión posterior, inicia un explorador web, ve a la [página de descarga de Git para Windows](https://gitforwindows.org/) e instálalo.
- Si aún no tienes instalado Visual Studio Code, ve a la [página de descarga de Visual Studio Code](https://code.visualstudio.com/) desde la ventana del explorador, descarga e instala esta aplicación.
- Si aún no tienes instalada la extensión C# de Visual Studio, ve a la [página de instalación de la extensión C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) en la ventana del explorador e instala la extensión.

## Introducción al laboratorio

Azure DevOps admite dos tipos de control de versiones, Git y Control de versiones de Team Foundation (TFVC). Esta es una introducción rápida de los dos sistemas de control de versiones:

- **Control de versiones de Team Foundation (TFVC):** es un sistema de control de versiones centralizado. Normalmente, los miembros del equipo solo tienen una versión de cada archivo en sus equipos de desarrollo. Los datos históricos se conservan únicamente en el servidor. Las bifurcaciones se basan en las rutas de acceso y se crean en el servidor.

- **Git:** es un sistema de control de versiones distribuido. Los repositorios de Git pueden residir en el entorno local (en la máquina de un desarrollador). Cada desarrollador tiene una copia del repositorio de origen en su máquina de desarrollo. Los desarrolladores pueden confirmar cada conjunto de cambios en sus equipos de desarrollo y realizar las operaciones de control de versiones en forma de historial para compararlos sin necesidad de una conexión de red.

Git es el proveedor de control de versiones predeterminado para los nuevos proyectos. Debe usar Git para el control de versiones en los proyectos a menos que necesite características centralizadas de control de versiones en TFVC.

En este laboratorio, aprenderá a establecer un repositorio de Git local, que se puede sincronizar fácilmente con un repositorio de Git centralizado en Azure DevOps. Además, obtendrá información sobre la compatibilidad con la creación de ramas y la combinación de Git. Usará Visual Studio Code, pero los mismos procesos se aplican al uso de cualquier cliente compatible con Git.

## Objetivos

Después de completar este laboratorio, podrá:

- Clonar un repositorio existente.
- Guardar el trabajo con confirmaciones.
- Revisar el historial de cambios.
- Trabajar con ramas mediante Visual Studio Code.

## Tiempo estimado: 60 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y elige **Scrum** en la lista desplegable **Proceso del elemento de trabajo**. Haga clic en **Crear**.

    ![Crear proyecto](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/create-project.png)

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar**. En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL https://github.com/MicrosoftLearning/eShopOnWeb.git y haz clic en **Importar**:

    ![Importar repositorio](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/import-repo.png)

2. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** tiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **.azure** contiene la infraestructura de Bicep&ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - La carpeta **.github** contiene definiciones de flujo de trabajo de GitHub YAML.
    - La carpeta **src** contiene el sitio web .NET 7 que se usa en los escenarios de laboratorio.

#### Tarea 3: configurar Git y Visual Studio Code

En esta tarea, instalarás y configurarás Git y Visual Studio Code, incluida la configuración del asistente de credenciales de Git para almacenar de forma segura las credenciales de Git que usas para comunicarte con Azure DevOps. Si ya has implementado estos requisitos previos, puedes ir directamente a la siguiente tarea.

1. En el equipo del laboratorio, inicia **Visual Studio Code**.
2. En la interfaz de Visual Studio Code, en el menú principal, selecciona **Terminal \| Nuevo terminal** para abrir el panel **TERMINAL**.
3. Asegúrate de que el terminal actual ejecute **PowerShell** comprobando si la lista desplegable de la esquina superior derecha del panel **TERMINAL** muestre **1: powershell**

    > **Nota**: para cambiar el shell de terminal actual a **PowerShell**, haz clic en la lista desplegable de la esquina superior derecha del panel **TERMINAL** y haz clic en **Seleccionar shell predeterminado**. En la parte superior de la ventana de Visual Studio Code, selecciona tu shell **Windows PowerShell** de terminal preferido y haz clic en signo más a la derecha de la lista desplegable para abrir un nuevo terminal con el el shell predeterminado seleccionado.

4. En el panel **TERMINAL**, ejecuta el siguiente comando para configurar el asistente de credenciales.

    ```git
    git config --global credential.helper wincred
    ```

5. En el panel **TERMINAL**, ejecuta los siguientes comandos para configurar un nombre de usuario y una dirección de correo para confirmaciones de Git (reemplaza los marcadores de posición entre llaves por el nombre de usuario y el correo electrónico preferidos, lo que elimina los símbolos < y >):

    ```git
    git config --global user.name "<John Doe>"
    git config --global user.email <johndoe@example.com>
    ```

### Ejercicio 1: clonación de un repositorio existente

En este ejercicio, usarás Visual Studio Code para clonar el repositorio de Git que has aprovisionado como parte del ejercicio anterior.

#### Tarea 1: clonar un repositorio existente

En esta tarea, aprenderás a clonar un repositorio de Git con Visual Studio Code.

1. Ve al explorador web que muestra la organización de Azure DevOps con el proyecto **eShopOnWeb** que has generado en el ejercicio anterior.
2. En el panel de navegación vertical del portal de Azure DevOps, selecciona el icono **Repos**.

3. En la esquina superior derecha del panel del repositorio **eShopOnWeb**, haz clic en **Clonar**.

    ![Clonación del repositorio de Git](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/clone-repo.png)

    > **Nota**: la obtención de una copia local de un repo de Git se denomina *clonación*. Cada herramienta de desarrollo estándar admite esto y podrá conectarse a Azure Repos para extraer las fuentes más recientes para trabajar.

4. En el panel **Clonar repositorio**, con la opción de línea de comandos **HTTPS** seleccionada, haz clic en el botón **Copiar al portapapeles** junto a la dirección URL de clonación del repo.

    > **Nota**: puedes usar esta dirección URL con cualquier herramienta compatible con Git para obtener una copia del código base.

5. Cierra el panel **Clonar repositorio**.
6. En el equipo de laboratorio, cambia a **Visual Studio Code**.
7. Haz clic en el encabezado del menú **Ver** y, en el menú desplegable, haz clic en **Paleta de comandos**.

    > **Nota**: la paleta de comandos ofrece una manera fácil y cómoda de acceder a una amplia variedad de tareas, incluidas las implementadas como extensiones de terceros. También puedes usar el método abreviado de teclado **Ctrl+Shift+P** o **F1** para abrirla.

8. En el símbolo de la paleta de comandos, ejecuta el comando **Git: Clone**.

    ![Paleta de comandos de VS Code](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/vscode-command.png)

    > **Nota**: para ver todos los comandos pertinentes, puedes empezar escribiendo **Git**.

9. En el cuadro de texto **Proporcionar dirección URL del repositorio o elegir un origen de repositorio**, pega la URL de clonación del repo que has copiado en esta tarea y presiona **Entrar**.
10. En el cuadro de diálogo **Seleccionar carpeta**, ve a la unidad C:, crea una carpeta denominada **Git**, selecciónala y haz clic en **Seleccionar ubicación del repositorio**.
11. Cuando se solicite, inicia sesión en tu cuenta de Azure DevOps.
12. Una vez completado el proceso de clonación, cuando se solicite, en Visual Studio Code, haz clic en **Abrir** para abrir el repositorio clonado.

    > **Nota**: puedes omitir las advertencias que recibas sobre problemas para cargar el proyecto. Es posible que la solución no esté en el estado adecuado para una compilación, pero nos centraremos en trabajar con Git, por lo que no es necesario compilar el proyecto.

### Ejercicio 2: almacenamiento del trabajo con confirmaciones

En este ejercicio, aprenderás varios escenarios que implican el uso de Visual Studio Code para almacenar provisionalmente y confirmar cambios.

Al realizar cambios en los archivos, Git los registrará en el repositorio local. Puedes seleccionar los cambios que quieres confirmar al almacenarlos provisionalmente. Las confirmaciones siempre se realizan en el repositorio de Git local, por lo que no tienes que preocuparte de que la confirmación sea perfecta o esté lista para compartir con otros usuarios. Puedes realizar más confirmaciones a medida que trabajas y enviar los cambios a otras personas cuando estén listos para compartirse.

Las confirmaciones de Git constan de lo siguiente:

- El archivo o los archivos modificados en la confirmación. Git mantiene el contenido de todos los cambios de archivos del repo en las confirmaciones. Esto permite trabajar rápidamente y combinar de forma inteligente.
- Una referencia a la confirmación principal. Git administra el historial de código mediante estas referencias.
- Mensaje que describe una confirmación. Este mensaje se envía a Git al crear la confirmación. Este mensaje debe ser descriptivo pero breve.

#### Tarea 1: confirmar cambios

En esta tarea, usarás Visual Studio Code para confirmar los cambios.

1. En la ventana de Visual Studio Code, en la parte superior de la barra de herramientas vertical, selecciona la pestaña **EXPLORER**, ve al archivo **/eShopOnWeb/src/Web/Program.cs** y selecciónalo. Esto mostrará el contenido automáticamente en el panel de detalles.
2. Agrega las siguientes líneas después del comentario:

    ```csharp
    // My first change
    ```

    > **Nota**: el contenido del comentario no es tan importante, ya que el objetivo es hacer un cambio.

3. Presione **CTRL+S** para guardar el cambio.
4. En la ventana de Visual Studio Code, selecciona la pestaña **CONTROL DE CÓDIGO FUENTE** para comprobar que Git ha reconocido el cambio más reciente en el archivo que reside en el clon local del repositorio de Git.
5. Con la pestaña **CONTROL DE CÓDIGO FUENTE** seleccionada, en la parte superior del panel, en el cuadro de texto, escribe **Mi confirmación** como mensaje de confirmación y presiona **Ctrl+Entrar** para confirmarlo en el equipo local.

    ![Primera confirmación](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/first-commit.png)

6. Si debes almacenar de manera automática y provisional los cambios, y confirmarlos directamente, haz clic en **Siempre**.

    > **Nota**: analizaremos **el almacenamiento provisional** más adelante en el laboratorio.

7. En la esquina inferior izquierda de la ventana de Visual Studio Code, a la derecha de la etiqueta **main**, observa el icono **Sincronizar cambios** de un círculo con dos flechas verticales que apuntan en las direcciones opuestas y el número **1** junto a la flecha que apunta hacia arriba. Haz clic en el icono y, si el sistema te pregunta si quieres continuar, haz clic en **Aceptar** para insertar y extraer confirmaciones hacia el elemento de **origin/main** y desde este.

#### Tarea 2: revisar confirmaciones

En esta tarea, usarás el portal de Azure DevOps para revisar las confirmaciones.

1. Ve a la ventana del explorador web donde aparece la interfaz de Azure Portal.
2. En el panel de navegación vertical del portal de Azure DevOps, en la sección **Repos**, selecciona **Confirmaciones**.
3. Comprueba que la confirmación aparezca en la parte superior de la lista.

    ![Confirmaciones del repo de ADO](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/ado-commit.png)

#### Tarea 3: cambiar fases

En esta tarea, descubrirás el uso de cambios de almacenamiento provisional mediante Visual Studio Code. Los cambios de almacenamiento provisional permiten agregar de forma selectiva ciertos archivos a una confirmación al pasar los cambios realizados en otros archivos.

1. Vuelva a la ventana de **Visual Studio Code**.
2. Actualiza la clase abierta **Program.cs** cambiando el primer comentario con el siguiente contenido y guardando el archivo.

    ```csharp
        //My second change
    ```

3. En la ventana de Visual Studio Code, vuelve a la pestaña **EXPLORER**, ve al archivo **/eShopOnWeb/src/Web/Constants.cs** y selecciónalo. Esto mostrará el contenido automáticamente en el panel de detalles.
4. Agrega un comentario en la primera línea del archivo **Constants.cs** y guarda.

    ```csharp
    // My third change
    ```

5. En la ventana de Visual Studio Code, cambia a la pestaña **CONTROL DE CÓDIGO FUENTE**, mantén el puntero del mouse sobre la entrada **Program.cs** y haz clic en el signo más a la derecha de esa entrada.

    > **Nota**: esto solo establece el cambio en el archivo **Program.cs**, únicamente para la confirmación y sin **Constants.cs**.

6. Con la pestaña **CONTROL DE CÓDIGO FUENTE** seleccionada, en la parte superior del panel, en el cuadro de texto, escribe **Comentarios agregados** como mensaje de confirmación.

    ![Cambios almacenados provisionalmente](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/staged-changes.png)

7. En la parte superior de la pestaña **CONTROL DE CÓDIGO FUENTE**, haz clic en los puntos suspensivos. En el menú desplegable, selecciona **Confirmar** y, en el menú en cascada, selecciona **Confirmar elementos almacenados provisionalmente**.
8. En la esquina inferior izquierda de la ventana de Visual Studio Code, haz clic en el botón **Sincronizar cambios** para sincronizar los cambios confirmados con el servidor y, si el sistema te pregunta si quieres continuar, haz clic en **Aceptar** para insertar y extraer confirmaciones hacia los elementos **origin/main** y desde ellos.

    > **Nota:** ten en cuenta que, dado que solo se ha confirmado el cambio preconfigurado, el otro cambio aún no se ha sincronizado.

### Ejercicio 3: revisión del historial

En este ejercicio, usarás el portal de Azure DevOps para revisar el historial de confirmaciones.

Git usa la información de referencia primaria almacenada en cada confirmación para administrar un historial completo de tu desarrollo. Puedes revisar fácilmente este historial de confirmaciones para averiguar cuándo se hicieron cambios en los archivos y determinar las diferencias entre las versiones del código mediante el terminal o desde una de las extensiones de Visual Studio Code disponibles. También puedes revisar los cambios mediante el portal de Azure DevOps.

El uso de Git de la característica **Ramas y combinaciones** funciona a través de solicitudes de incorporación de cambios, por lo que el historial de confirmaciones del desarrollo no necesariamente sigue una línea cronológica recta. Cuando se usa el historial para comparar versiones, piensa en los cambios de archivo entre dos confirmaciones, no entre dos momentos diferentes. Un cambio reciente en un archivo de la rama principal puede haber venido de una confirmación creada hace dos semanas en una rama de características que se combinó ayer.

#### Tarea 1: comparar archivos

En esta tarea, conocerás el historial de confirmaciones mediante el portal de Azure DevOps.

1. Con la pestaña **CONTROL DE CÓDIGO FUENTE** de la ventana de Visual Studio Code abierta, selecciona **Constants.cs** que representa la versión no almacenada provisionalmente del archivo.

    ![Comparación de archivos](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/file-comparison.png)

    > **Nota**: se abre una vista de comparación que te permite localizar fácilmente los cambios realizados. En este caso, es solo un comentario.

2. Cambia a la ventana del explorador web que muestra el panel **Confirmaciones** del portal de **Azure DevOps** para revisar las ramas de origen y las combinaciones. Se proporciona una manera cómoda de visualizar cuándo y cómo se realizaron los cambios en el origen.
3. Desplázate hacia abajo hasta la entrada **Mi confirmación** (insertada anteriormente) y mantén el puntero del mouse sobre esta hasta ver los puntos suspensivos a la derecha.
4. Haz clic en los puntos suspensivos y, en el menú desplegable, selecciona **Examinar archivos** y revisa los resultados.

    ![Exploración de la confirmación](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/commit-browse.png)

    > **Nota**: esta vista representa el estado del origen correspondiente a la confirmación, lo que te permite revisar y descargar los archivos de origen.

### Ejercicio 4: trabajo con ramas

En este ejercicio, conocerás los escenarios que implican la administración de ramas con Visual Studio Code y el portal de Azure DevOps.

Puedes realizar tareas de administración en el repo de Git de Azure DevOps desde la vista **Ramas** de **Azure Repos** en el portal de Azure DevOps. También puedes personalizar la vista para hacer un seguimiento de las ramas que más te interesan para mantenerte al día de los cambios realizados por tu equipo.

Confirmar cambios en una rama no afectará a otras ramas, y puedes compartir ramas con otras personas sin necesidad de combinar los cambios en el proyecto principal. Puedes crear nuevas ramas para aislar los cambios de una característica o una corrección de errores de la rama principal y del trabajo restante. Dado que las ramas son ligeras, alternar entre ramas es rápido y fácil. Git no crea varias copias del origen cuando se trabaja con ramas, sino que usa la información de historial almacenada en las confirmaciones para volver a crear los archivos en una rama al empezar a trabajar en ella. El flujo de trabajo de Git debe crear y usar ramas para administrar características y correcciones de errores. El resto del flujo de trabajo de Git, como compartir código y revisarlo con solicitudes de incorporación de cambios, funciona mediante ramas. El aislamiento del trabajo en ramas te permite cambiar en qué estás trabajando con solo cambiar la rama actual.

#### Tarea 1: crear una nueva rama local en el repositorio clonado

En esta tarea, crearás una rama mediante Visual Studio Code.

1. En el equipo de laboratorio, cambia a **Visual Studio Code**.
2. Con la pestaña **CONTROL DE CÓDIGO FUENTE** seleccionada, en la esquina inferior izquierda de la ventana de Visual Studio Code, haz clic en **main**.
3. En la ventana emergente, selecciona **+ Crear nueva rama desde...**.

    ![Crear rama](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/create-branch.png)

4. En el cuadro de texto **Nombre de rama**, escribe **dev** para especificar la nueva rama y presiona **Entrar**.
5. En el cuadro de texto **Seleccionar una referencia para crear la rama "dev"**, selecciona **main** como la rama de referencia.

    > **Nota**: aquí cambias automáticamente a la rama **dev**.

#### Tarea 2: eliminar una rama

En esta tarea, usarás Visual Studio Code para trabajar con una rama creada en la tarea anterior.

Git realiza un seguimiento de la rama en la que estás trabajando y se asegura de que, cuando revises una rama, los archivos coincidan con la confirmación más reciente. Las ramas permiten trabajar con varias versiones del código fuente en el mismo repositorio de Git local al mismo tiempo. Puedes usar Visual Studio Code para publicar, revisar y eliminar ramas.

1. En la ventana **Visual Studio Code**, con la pestaña **CONTROL DE CÓDIGO FUENTE** seleccionada, en la esquina inferior izquierda de la ventana de Visual Studio Code, haz clic en el icono **Publicar cambios** (a la derecha de la etiqueta **dev** que representa la rama recién creada).
2. Ve a la ventana del explorador web que muestra el panel **Confirmaciones** del portal de **Azure DevOps** y selecciona **Ramas**.
3. En la pestaña **Mías** del panel **Ramas**, comprueba que esté **dev** en la lista de ramas.
4. Mantén el puntero del mouse sobre la entrada de la rama **dev** para ver los puntos suspensivos a la derecha.
5. Haz clic en los puntos suspensivos y, en el menú emergente, selecciona **Eliminar rama** y, cuando el sistema te pida confirmación, haz clic en **Eliminar**.

    ![Eliminar rama](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/delete-branch.png)

6. Vuelve a la ventana de **Visual Studio Code** y, con la pestaña ** CONTROL DE CÓDIGO FUENTE** seleccionada, en la esquina inferior izquierda de la ventana de Visual Studio Code, haz clic en la entrada **dev**. Verás las ramas existentes en la parte superior de la ventana de Visual Studio Code.
7. Comprueba que haya dos ramas **dev** en la lista.

    > **Nota**: la rama local (**dev**) aparece allí porque su existencia no se ve afectada por la eliminación de la rama en el repositorio remoto. El servidor (**origin/dev**) aparece allí porque no se ha eliminado.

8. En la lista de ramas, selecciona la rama **main** para revisarla.
9. Presiona **Ctrl+Shift+P** para abrir la **paleta de comandos**.
10. En el símbolo del sistema **Paleta de comandos**, empieza a escribir **Git: Eliminar** y selecciona **Git: Eliminar rama** cuando esté visible.
11. Selecciona la entrada **dev** en la lista de ramas que se van a eliminar.
12. En la esquina inferior izquierda de la ventana de Visual Studio Code, vuelve a hacer clic en la entrada **main**. Verás las ramas existentes en la parte superior de la ventana de Visual Studio Code.
13. Comprueba que la rama **dev** local ya no aparezca en la lista, pero que la **origin/dev** remota todavía esté ahí.
14. Presiona **Ctrl+Shift+P** para abrir la **paleta de comandos**.
15. En el símbolo de la **Paleta de comandos**, empieza a escribir **Captura de Git** y selecciona **Captura de Git (Eliminar)** cuando esté visible.

    > **Nota**: este comando actualizará las ramas de origen en la instantánea local y eliminará las que ya no estén allí.

    > **Nota**: para comprobar exactamente qué hacen estas tareas, selecciona la ventana **Salida** en la parte inferior derecha de la ventana de Visual Studio Code. Si no ves los registros de Git en la consola de salida, asegúrate de seleccionar **Git** como el origen.

16. En la esquina inferior izquierda de la ventana de Visual Studio Code, vuelve a hacer clic en la entrada **main**.
17. Comprueba que la rama **origin/dev** ya no aparezca en la lista de ramas.

#### Tarea 3: restaurar una rama

En esta tarea, usarás el portal de Azure DevOps para restaurar la rama que has eliminado en la tarea anterior.

1. Ve al explorador web que muestra la pestaña **Mías** del panel **Ramas** en el portal de Azure DevOps.
2. En la pestaña **Mías** del panel **Ramas**, selecciona la pestaña **Todas**.
3. En la pestaña **Todas** del panel **Ramas**, en el cuadro de texto **Buscar nombre de rama**, escribe **dev**.
4. Revisa la sección **Ramas eliminadas** que contiene la entrada que representa la rama eliminada recientemente.
5. En la sección **Ramas eliminadas**, mantén el puntero del mouse sobre la entrada de rama **dev** para ver los puntos suspensivos a la derecha.
6. Haz clic en los puntos suspensivos y, en el menú emergente, selecciona **Restaurar rama**.

    ![restaurar rama](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/restore-branch.png)

    > **Nota**: puedes usar esta funcionalidad para restaurar una rama eliminada si conoces su nombre exacto.

#### Tarea 4: directivas de rama

En esta tarea, usarás el portal de Azure DevOps para agregar directivas a la rama principal y solo permitir cambios mediante solicitudes de incorporación de cambios que cumplan las directivas definidas. Deberías asegurarte de que los cambios de una rama se revisen antes de combinarlos.

Para simplificar, trabajaremos directamente en el editor de repositorios del explorador web (directamente en el origen), en lugar de usar el clon local en VS Code (recomendado para escenarios reales).

1. Ve al explorador web que muestra la pestaña **Mías** del panel **Ramas** en el portal de Azure DevOps.
2. En la pestaña **Mías** del panel **Ramas**, mantén el puntero del mouse sobre la entrada de rama **main** para ver los puntos suspensivos a la derecha.
3. Haz clic en los puntos suspensivos y, en el menú emergente, selecciona **Directivas de ramas**.

    ![Directivas de ramas](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/branch-policies.png)

4. En la pestaña **main** de la configuración del repositorio, habilita la opción **Requerir un número mínimo de revisores**. Agrega **1** revisor y activa la casilla **Permitir que los solicitantes aprueben sus propios cambios** (ya que eres el único usuario del proyecto para el laboratorio)
5. En la pestaña **main** de la configuración del repositorio, habilita la opción **Buscar elementos de trabajo vinculados** y déjala configurada como **Obligatorio**.

    ![Configuración de directivas](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/policy-settings.png)

#### Tarea 5: prueba de la directiva de ramas

En esta tarea, usarás el portal de Azure DevOps para probar la directiva y crear la primera solicitud de incorporación de cambios.

1. En el panel de navegación vertical del portal de Azure DevOps, en **Repos>Archivos**, asegúrate de que la rama **main** esté seleccionada (lista desplegable anterior con el contenido).
2. Para asegurarte de que las directivas funcionan, intenta realizar un cambio y confirmarlo en la rama **main**, ve al archivo **/eShopOnWeb/src/Web/Program.cs** y selecciónalo. Esto mostrará el contenido automáticamente en el panel de detalles.
3. Agrega las siguientes líneas después del comentario:

    ```csharp
    // Testing main branch policy
    ```

4. Haz clic en **Confirmar > Confirmar**. Verás una advertencia: los cambios realizados en la rama principal solo se pueden hacer mediante una solicitud de incorporación de cambios.

    ![Confirmación denegada de la directiva](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/policy-denied.png)

5. Haz clic en **Cancelar** para omitir la confirmación.

#### Tarea 6: trabajar con solicitudes de incorporación de cambios

En esta tarea, usarás el portal de Azure DevOps para crear una solicitud de incorporación de cambios mediante la rama **dev** para combinar un cambio dentro de la rama **principal** protegida. Un elemento de trabajo de Azure DevOps se vinculará a los cambios para hacer un seguimiento del trabajo pendiente con la actividad del código.

1. En el panel de navegación vertical del portal de Azure DevOps, en la sección **Paneles**, selecciona **Elementos de trabajo **.
2. Haz clic en **+ Nuevo elemento de trabajo > Elemento de trabajo pendiente**. En el campo de título, escribe **Probar mi primera PR** y haz clic en **Guardar**.
3. Ahora vuelve al panel de navegación vertical del portal de Azure DevOps y, en **Repos>Archivos**, asegúrate de que esté seleccionada la rama **dev**.
4. Ve al archivo **/eShopOnWeb/src/Web/Program.cs** y realiza el siguiente cambio en la primera línea:

    ```csharp
    // Testing my first PR
    ```

5. Haz clic en **Confirmar > Confirmar** (deja el mensaje de confirmación predeterminado). Esta vez la confirmación funciona, la rama **dev** no tiene directivas.
6. Aparecerá un mensaje que propone crear una solicitud de incorporación de cambios (ya que la rama **dev** ahora tiene más cambios, en comparación con **main**). Haz clic en **Crear solicitud de incorporación de cambios**.

    ![Crear una solicitud de cambios](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/create-pr.png)

7. En la pestaña **Nueva solicitud de incorporación de cambios**, deja los valores predeterminados y haz clic en **Crear**.
8. La solicitud de incorporación de cambios mostrará algunos requisitos con errores o pendientes, en función de las directivas aplicadas a nuestra rama **main** de destino.
    - Los cambios propuestos deben tener un elemento de trabajo vinculado
    - Al menos 1 usuario debe revisar y aprobar los cambios.

9. En las opciones del lado derecho, haz clic en el botón **+** situado junto a **Elementos de trabajo**. Haz clic en el elemento de trabajo creado anteriormente para vincularlo a la solicitud de incorporación de cambios. Verás uno de los cambios de estado de los requisitos.

    ![Vincular elemento de trabajo](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/link-wit.png)

10. A continuación, abre la pestaña **Archivos** para revisar los cambios propuestos. En una solicitud de incorporación de cambios más completa, podrías revisar los archivos uno por uno (marcado como revisado) y abrir comentarios en las líneas que se pueden prestar a confusión (pasa el mouse sobre el número de línea y verás la opción de publicar un comentario).
11. Vuelve a la pestaña **Información general** y, en la parte superior derecha, haz clic en **Aprobar**. Todos los requisitos se verán en verde. Ahora puedes hacer clic en **Completar**.
12. En la pestaña **Completar solicitud de incorporación de cambios**, se proporcionarán varias opciones antes de completar la combinación:
    - **Tipo de combinación**: hay 4 tipos de combinación, y puedes revisarlos [aquí](https://learn.microsoft.com/azure/devops/repos/git/complete-pull-requests?view=azure-devops&tabs=browser#complete-a-pull-request) o ver las animaciones. Elige **Combinar (sin avance rápido)**.
    - **Opciones posteriores a la finalización**:
        - Activa **Completar elemento de trabajo asociado...**. Se moverá el PBI asociado al estado **Listo**.

13. Haz clic en **Completar combinación**

#### Tarea 7: aplicar etiquetas

El equipo del producto ha decidido que la versión actual del sitio debe publicarse como v1.1.0-beta.

1. En el panel de navegación vertical del portal de Azure DevOps, en la sección **Repos**, selecciona **Etiquetas**.
2. En el panel **Etiquetas**, haz clic en **Nueva etiqueta**.
3. En el panel **Crear una etiqueta**, en el cuadro de texto **Nombre**, escribe **v1.1.0-beta**; en la lista desplegable **Basado en**, deja seleccionada la entrada **main**. En el cuadro de texto **Descripción**, escribe **Versión beta v1.1.0** y haz clic en **Crear**.

    > **Nota**: ahora has etiquetado el repositorio en esta versión (la confirmación más reciente se vincula a la etiqueta). Puedes etiquetar confirmaciones por diversos motivos, y Azure DevOps te permite editarlas y eliminarlas, así como administrar sus permisos.

### Ejercicio 5: Quitar directivas de rama

Al pasar por los diferentes laboratorios del curso en el orden en que se presentan, la directiva de rama configurada durante este laboratorio bloqueará los ejercicios en laboratorios futuros. Por lo tanto, queremos quitar las directivas de rama configuradas.

1. Desde la vista de proyecto de Azure DevOps **EShopOnWeb**, navegue hasta **Repos** y seleccione **Ramas**. Seleccione la pestaña **Mina** del panel **Ramas**.
2. En la pestaña **Mina** del panel **Ramas**, pase el puntero del ratón por encima de la entrada de la rama **principal** para que aparezca el símbolo de puntos suspensivos (...) a la derecha.
3. Haz clic en los puntos suspensivos y, en el menú emergente, selecciona **Directivas de ramas**.

    ![Configuración de directivas](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/policy-settings.png)

4. En la pestaña **principal** de la configuración del repositorio, desactive la opción para **Requerir un número mínimo de revisores**.
5. En la pestaña **principal** de la configuración del repositorio, desactive la opción de **Comprobar elementos de trabajo vinculados**.

    ![Directivas de ramas](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/images/branch-policies.png)

6. Ahora ha deshabilitado o quitado las directivas de rama de la rama principal.
    

## Revisar

En este laboratorio, has usado el portal de Azure DevOps para administrar ramas y repositorios.
