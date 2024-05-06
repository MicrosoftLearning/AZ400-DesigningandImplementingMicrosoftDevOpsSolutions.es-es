---
lab:
  title: "Habilitación de la integración continua con Azure\_Pipelines"
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Habilitación de la integración continua con Azure Pipelines

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization).

## Introducción al laboratorio

En este laboratorio, aprenderá a definir canalizaciones de compilación en Azure DevOps mediante YAML.
Las canalizaciones se usarán en dos escenarios:

- Como parte del proceso de validación de solicitudes de incorporación de cambios.
- Como parte de la implementación de integración continua.

## Objetivos

Después de completar este laboratorio, podrá:

- Incluya la validación de compilación como parte de una solicitud de incorporación de cambios.
- Configure la canalización de CI como código con YAML.

## Tiempo estimado: 45 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: (omitir si ya la has completado) crear y configurar el proyecto del equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asígnale al proyecto el nombre **eShopOnWeb** y deja los demás campos con los valores predeterminados. Haga clic en **Crear**.

#### Tarea 2: (omitir si ha terminado) Importar repositorio de Git eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar un repositorio**. Seleccione **Import** (Importar). En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL https://github.com/MicrosoftLearning/eShopOnWeb.git y haz clic en **Importar**:

1. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - La carpeta **.github** contiene definiciones de flujo de trabajo de GitHub de YAML.
    - La carpeta **src** contiene el sitio web de .NET que se usa en los escenarios de laboratorio.

### Ejercicio 1: Incluir la validación de compilación como parte de una solicitud de incorporación de cambios

En este ejercicio, incluirás la validación de compilación para validar una solicitud de incorporación de cambios.

#### Tarea 1: Importación de la definición de compilación de YAML

En esta tarea, importarás la definición de compilación de YAML que se usará como directiva de rama para validar las solicitudes de incorporación de cambios.

Empecemos importando la canalización de compilación denominada [eshoponweb-ci-pr.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci-pr.yml).

1. Ve a **Canalizaciones>Canalizaciones**
1. Haz clic en el botón **Crear canalización** o **Nueva canalización.**
1. Selecciona **Git de Azure Repos (YAML)**
1. Selecciona el repositorio **eShopOnWeb**
1. Selecciona **Archivo YAML de Azure Pipelines existente**
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-ci-pr.yml**, y haga clic en **Continuar**

    La canalización de compilación está formada por las siguientes tareas:
    - **DotNet Restore**: con la restauración de paquetes NuGet, puedes instalar todas las dependencias del proyecto sin tener que almacenarlas en el control de código fuente.
    - **DotNet Build**: compila un proyecto y todas sus dependencias.
    - **DotNet Test**: controlador de prueba de .Net para ejecutar pruebas unitarias.
    - **DotNet Publish**: publica la aplicación y sus dependencias en una carpeta para la implementación en un sistema de hospedaje. En este caso, es **Build.ArtifactStagingDirectory**.

1. Haz clic en el botón **Guardar** para guardar la definición de canalización
1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y haz clic en la canalización creada recientemente. Haga clic en los puntos suspensivos y, después, en la opción **Cambiar nombre/mover**. Asígnale el nombre **eshoponweb-ci-pr** y haz clic en **Guardar**.

#### Tarea 2: Directivas de rama

En esta tarea, agregarás directivas a la rama main y solo permitirás cambios mediante solicitudes de incorporación de cambios que cumplan las directivas definidas. Deberías asegurarte de que los cambios de una rama se revisen antes de combinarlos.

1. Ve a la sección **Repos>Ramas**.
1. En la pestaña **Mías** del panel **Ramas**, mantén el puntero del mouse sobre la entrada de rama **main** para ver los puntos suspensivos a la derecha.
1. Haz clic en los puntos suspensivos y, en el menú emergente, selecciona **Directivas de ramas**.
1. En la pestaña **main** de la configuración del repositorio, habilita la opción **Requerir un número mínimo de revisores**. Agrega **1** revisor y activa la casilla **Permitir que los solicitantes aprueben sus propios cambios** (ya que eres el único usuario del proyecto para el laboratorio)
1. En la pestaña **main** de la configuración del repositorio, en la sección **Validación de compilación**, haz clic en**+**(Agregar nueva directiva de compilación) y, en la lista **Canalización de compilación**, selecciona **eshoponweb-ci-pr** y haz clic en **Guardar**.

#### Tarea 3: Trabajar con solicitudes de incorporación de cambios

En esta tarea, usarás el portal de Azure DevOps para crear una solicitud de incorporación de cambios, empleando una nueva rama para combinar un cambio en la rama **main** protegida.

1. Ve a la sección **Repos** en la navegación eShopOnWeb y haz clic en **Ramas**.
1. Crear una nueva rama llamada **Feature01** basada en la rama **main**.
1. Haz clic en **Feature01** y ve al archivo **/eShopOnWeb/src/Web/Program.cs** como parte de la rama **Feature01**
1. Haz clic en el botón **Editar** en la esquina superior derecha.
1. Realiza el siguiente cambio en la primera línea:

    ```csharp
    // Testing my PR
    ```

1. Haz clic en **Confirmar > Confirmar** (deja el mensaje de confirmación predeterminado).
1. Aparecerá un mensaje proponiendo crear una solicitud de incorporación de cambios (ya que la rama **Feature01** ahora está por delante en los cambios, en comparación con la rama **main**). Haz clic en **Crear solicitud de incorporación de cambios**.
1. En la pestaña **Nueva solicitud de incorporación de cambios**, deja los valores predeterminados y haz clic en **Crear**.
1. La solicitud de incorporación de cambios mostrará algunos requisitos pendientes en función de las directivas aplicadas a la rama **main** de destino.
    - Al menos 1 usuario debe revisar y aprobar los cambios.
    - Validación de compilación, verás que la compilación **eshoponweb-ci-pr** se desencadenó automáticamente.

1. Después de que todas las validaciones se realicen correctamente, haz clic con el botón derecho en **Aprobar** en la parte superior. Ahora, en la lista desplegable **Establecer autocompletar**, puedes hacer clic en **Completar**.
1. En la pestaña **Completar solicitud de incorporación de cambios**, haz clic en **Completar la fusión**

### Ejercicio 2: Configurar la canalización de CI como código con YAML

En este ejercicio, configurarás la canalización de CI como código con YAML.

#### Tarea 1: Importación de la definición de compilación de YAML para CI

En esta tarea, agregarás la definición de compilación de YAML que se usará para implementar la integración continua.

Empecemos importando la canalización de CI denominada [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Ve a **Canalizaciones>Canalizaciones**.
1. Haz clic en el botón **Nueva canalización**.
1. Seleccione **GIT de Azure Repos** (YAML).
1. Selecciona el repositorio **eShopOnWeb**.
1. Selecciona el **archivo YAML de Azure Pipelines existente**.
1. Seleccione la rama **principal** y el archivo **/.ado/eshoponweb-ci.yml**, y haga clic en **Continuar**.

    La definición de CI comprende las tareas siguientes:
    - **DotNet Restore**: con la restauración de paquetes NuGet, puedes instalar todas las dependencias del proyecto sin tener que almacenarlas en el control de código fuente.
    - **DotNet Build**: compila un proyecto y todas sus dependencias.
    - **DotNet Test**: controlador de prueba de .Net para ejecutar pruebas unitarias.
    - **DotNet Publish**: publica la aplicación y sus dependencias en una carpeta para la implementación en un sistema de hospedaje. En este caso, es **Build.ArtifactStagingDirectory**.
    - **Publish Artifact - Website**: publica el artefacto de la aplicación (creado en el paso anterior) y habilítalo como artefacto de canalización.
    - **Publish Artifact - Bicep**: publica el artefacto de infraestructura (archivo de Bicep) y habilítalo como artefacto de canalización.

#### Tarea 2: habilitar la integración continua

La definición de canalización de compilación predeterminada no habilita la integración continua.

1. Haz clic en el botón **Editar** en la esquina superior derecha.
1. Ahora debes reemplazar el **desencadenador #:** y **# - líneas principales** por el código siguiente:

    ```YAML
    trigger:
      branches:
        include:
        - main
      paths:
        include:
        - src/web/*
    ```

    Esto desencadenará automáticamente la canalización de compilación si se realiza algún cambio en la rama principal y en el código de la aplicación web (la carpeta src/web).

    Dado que has habilitado las directivas de rama, debes pasar por una solicitud de incorporación de cambios para actualizar el código.

1. Haz clic en el botón **Guardar** (no en **Guardar y ejecutar**) para guardar la definición de la canalización.
1. Seleccione **Crear una rama nueva para este commit**.
1. Mantén activado el nombre de rama predeterminado y la opción **Iniciar solicitud de incorporación de cambios**.
1. Haga clic en **Guardar**.
1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización. Ve a **Canalizaciones>Canalizaciones** y haz clic en la canalización creada recientemente. Haga clic en los puntos suspensivos y, después, en la opción **Cambiar nombre/mover**. Asígnale el nombre **eshoponweb-ci** y haz clic en **Guardar**.
1. Vaya a **Repositorios > Solicitudes de incorporación de cambios**.
1. Haga clic en la solicitud de incorporación de cambios **"Actualizar eshoponweb-ci.yml para Azure Pipelines"**.
1. Después de que todas las validaciones se realicen correctamente, haz clic con el botón derecho en **Aprobar** en la parte superior. Ahora puedes hacer clic en **Completar**.
1. En la pestaña **Completar solicitud de incorporación de cambios**, haz clic en **Completar la fusión**

#### Tarea 3: probar la canalización de CI/CD

En esta tarea, crearás una solicitud de incorporación de cambios mediante una nueva rama para combinar un cambio en la rama **principal** protegida, y desencadenarás la canalización de CI automáticamente.

1. Vaya a la sección **Repos**.
1. Cree una rama denominada **Feature02** basada en la rama **principal**.
1. Haga clic en la nueva rama **Feature02**.
1. Vaya al archivo **/eShopOnWeb/src/Web/Program.cs** y haga clic en **Editar** en la parte superior derecha.
1. Quita la primera línea:

    ```csharp
    // Testing my PR
    ```

1. Haz clic en **Confirmar > Confirmar** (deja el mensaje de confirmación predeterminado).
1. Aparecerá un mensaje que propone crear una solicitud de incorporación de cambios (ya que la rama **Feature02** ahora figura antes que **main** en los cambios).
1. Haz clic en **Crear solicitud de incorporación de cambios**.
1. En la pestaña **Nueva solicitud de incorporación de cambios**, deja los valores predeterminados y haz clic en **Crear**.
1. La solicitud de incorporación de cambios mostrará algunos requisitos pendientes en función de las directivas aplicadas a la rama **main** de destino.
1. Después de que todas las validaciones se realicen correctamente, haz clic con el botón derecho en **Aprobar** en la parte superior. Ahora, en la lista desplegable **Establecer autocompletar**, puedes hacer clic en **Completar**.
1. En la pestaña **Completar solicitud de incorporación de cambios**, haz clic en **Completar la fusión**
1. Vuelve a **Canalizaciones>Canalizaciones**, y verás que la compilación **eshoponweb-ci** se desencadenó automáticamente después de combinar el código.
1. Haz clic en la compilación **eshoponweb-ci** y selecciona la última ejecución.
1. Después de su correcta ejecución, haz clic en **Relacionado > Publicado** para controlar los artefactos publicados:
    - Bicep: el artefacto de la infraestructura.
    - Sitio web: el artefacto de la aplicación.

## Revisar

En este laboratorio, has habilitado la validación de solicitudes de incorporación de cambios mediante una definición de compilación y una canalización de CI configurada como código con YAML en Azure DevOps.
