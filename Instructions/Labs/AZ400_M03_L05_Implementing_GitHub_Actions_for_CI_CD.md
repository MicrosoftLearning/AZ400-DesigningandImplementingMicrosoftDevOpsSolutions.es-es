---
lab:
  title: Implementación de Acciones de GitHub para CI/CD
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Implementación de Acciones de GitHub para CI/CD

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps.](https://docs.microsoft.com/azure/devops/server/compatibility)

- Identifique una suscripción de Azure existente o cree una.

- Compruebe que tiene una cuenta de Microsoft o de Microsoft Entra con el rol Propietario o Colaborador en la suscripción a Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

- **Si aún no tienes una cuenta de GitHub** que puedas usar para este laboratorio, sigue las instrucciones disponibles en [Registro para obtener una nueva cuenta de GitHub](https://github.com/join) para crear una.

## Introducción al laboratorio

En este laboratorio, aprenderás a implementar un flujo de trabajo de Acciones de GitHub que implementa una aplicación web de Azure.

## Objetivos

Después de completar este laboratorio, podrá:

- Implemente un flujo de trabajo de Acciones de GitHub para CI/CD.
- Explicar las características básicas de los flujos de trabajo de Acciones de GitHub

## Tiempo estimado: 40 minutos

## Instrucciones

### Ejercicio 0: Importar eShopOnWeb al repositorio de GitHub

En este ejercicio, importarás el código del repositorio [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) existente en tu propio repositorio privado de GitHub.

El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **.azure** contiene la infraestructura de Bicep&ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - La carpeta **.github** contiene definiciones de flujo de trabajo de GitHub YAML.
    - La carpeta **src** contiene el sitio web .NET 7 que se usa en los escenarios de laboratorio.

#### Tarea 1: Crear un repositorio público en GitHub e importar eShopOnWeb

En esta tarea, crearás un repositorio de GitHub público vacío e importarás el repositorio [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) existente.

1. En el equipo de laboratorio, inicia un explorador web, ve al [sitio web de GitHub](https://github.com/), inicia sesión con tu cuenta y haz clic en **New** para crear un nuevo repositorio.

    ![Crear repositorio](images/github-new.png)

2. En la página **Create a new repository**, haz clic en el vínculo **Import a repository** (debajo del título de página).

    > NOTA: También puedes abrir el sitio web de importación directamente en https://github.com/new/import

3. En la página **Import your project to GitHub**:

    | Campo | Value |
    | --- | --- |
    | Dirección URL clon del repositorio anterior| https://github.com/MicrosoftLearning/eShopOnWeb |
    | Propietario | Alias de tu cuenta |
    | Nombre del repositorio | eShopOnWeb |
    | Privacidad | **Pública** |

4. Haz clic en **Begin Import** y espera a que el repositorio esté listo.

5. En la página del repositorio, ve a **Settings**, haz clic en **Actions > General** y elige la opción **Allow all actions and reusable workflows**. Haga clic en **Guardar**.

    ![Habilitación de Acciones de GitHub](images/enable-actions.png)

### Ejercicio 1: Configurar el repositorio de GitHub y el acceso a Azure

En este ejercicio, crearás una entidad de servicio de Azure para autorizar a GitHub a acceder a la suscripción de Azure desde Acciones de GitHub. También configurarás el flujo de trabajo de GitHub que compilará, probará e implementará el sitio web en Azure.

#### Tarea 1: Crear una entidad de servicio de Azure y guardarla como secreto de GitHub

En esta tarea, crearás la entidad de servicio de Azure que usa GitHub para implementar los recursos deseados. Como alternativa, también podrías usar [OpenID connect en Azure](https://docs.github.com/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure), como mecanismo de autenticación sin secretos.

1. En tu equipo del laboratorio, abre Azure Portal en una ventana del explorador (https://portal.azure.com/).
2. En el portal, busca **Grupos de recursos** y haz clic en él.
3. Haz clic en **+ Crear** para crear un nuevo grupo de recursos para el ejercicio.
4. En la pestaña **Crear un grupo de recursos**, asigna el nombre siguiente al grupo de recursos: **rg-az400-eshoponweb-NAME** (reemplaza NAME por algún alias único). Haz clic en **Revisar y crear > Crear**.
5. En Azure Portal, abre **Cloud Shell** (junto a la barra de búsqueda).

    > NOTA: Si es la primera vez que abres Cloud Shell, debes configurar el [almacenamiento persistente](https://learn.microsoft.com/azure/cloud-shell/persisting-shell-storage).

6. Asegúrate de que el terminal se ejecuta en modo **Bash** y ejecuta el siguiente comando, reemplazando **SUBSCRIPTION-ID** y **RESOURCE-GROUP** por tus propios identificadores (ambos se pueden encontrar en la página **Información general** del grupo de recursos):

    `az ad sp create-for-rbac --name GH-Action-eshoponweb --role contributor --scopes /subscriptions/SUBSCRIPTION-ID/resourceGroups/RESOURCE-GROUP --sdk-auth`

    > NOTA: Asegúrate de que se escribe o pega como una sola línea.
    > NOTA: Este comando creará una entidad de servicio con acceso de colaborador al grupo de recursos creado antes. De este modo, nos aseguramos de que Acciones de GitHub solo tendrá los permisos necesarios para interactuar únicamente con este grupo de recursos (no con el resto de la suscripción).

7. El comando generará un objeto JSON; más adelante lo usarás como secreto de GitHub para el flujo de trabajo. Copia el archivo JSON. El archivo JSON contiene los identificadores usados para autenticarte en Azure en el nombre de una identidad de Microsoft Entra (entidad de servicio).

    ```JSON
        {
            "clientId": "<GUID>",
            "clientSecret": "<GUID>",
            "subscriptionId": "<GUID>",
            "tenantId": "<GUID>",
            (...)
        }
    ```
8. También debes ejecutar el siguiente comando para registrar el proveedor de recursos para **Azure App Service** que implementarás más adelante:
   ```bash
   az provider register --namespace Microsoft.Web
   ``` 
10. En una ventana del explorador, vuelve al repositorio de GitHub **eShopOnWeb**.
11. En la página del repositorio, ve a **Settings**, haz clic en **Secrets and variables > Actions**. Haz clic en **New repository secret**.
    - Nombre: **AZURE_CREDENTIALS**
    - Secret: **pega el objeto JSON copiado anteriormente** (GitHub puede mantener varios secretos con el mismo nombre, que usa la acción [azure/login](https://github.com/Azure/login)).

12. Haz clic en **Add secret**. Ahora Acciones de GitHub podrá hacer referencia a la entidad de servicio mediante el secreto del repositorio.

#### Tarea 2: Modificar y ejecutar el flujo de trabajo de GitHub

En esta tarea, modificarás el flujo de trabajo de GitHub determinado y lo ejecutarás para implementar la solución en tu propia suscripción.

1. En una ventana del explorador, vuelve al repositorio de GitHub **eShopOnWeb**.
2. En la página del repositorio, ve a **Code** y abre el siguiente archivo: **eShopOnWeb/.github/workflows/eshoponweb-cicd.yml**. Este flujo de trabajo define el proceso de CI/CD para el código de sitio web de .NET 7 especificado.
3. Quita la marca de comentario de la sección **en** (elimina "#"). El flujo de trabajo se desencadena con cada inserción en la rama principal y también ofrece desencadenadores manuales ("workflow_dispatch").
4. En la sección **env**, realiza los cambios siguientes:
    - Reemplaza **NAME** en la variable **RESOURCE-GROUP**. Debería ser el mismo grupo de recursos que creaste en pasos anteriores.
    - (Opcional) Puedes elegir la [región de Azure](https://azure.microsoft.com/explore/global-infrastructure/geographies) más cercana en **LOCATION**. Por ejemplo, "eastus", "eastasia", "westus", etc.
    - Reemplaza **YOUR-SUBS-ID** en **SUBSCRIPTION-ID**.
    - Reemplaza **NAME** en **WEBAPP-NAME** por algún alias único. Se usará para crear un sitio web único globalmente mediante Azure App Service.
5. Lee detenidamente el flujo de trabajo y los comentarios que se proporcionan para entender mejor.

6. Haz clic en **Start Commit** y **Commit Changes** dejando los valores predeterminados (cambiando la rama principal). El flujo de trabajo se ejecutará automáticamente.

#### Tarea 3: Revisar la ejecución del flujo de trabajo de GitHub

En esta tarea, revisarás la ejecución del flujo de trabajo de GitHub:

1. En una ventana del explorador, vuelve al repositorio de GitHub **eShopOnWeb**.
2. En la página del repositorio, ve a **Actions** y verás la configuración del flujo de trabajo antes de ejecutarse. Haga clic en él.

    ![Flujo de trabajo de GitHub en curso](images/gh-actions.png)

3. Espera a que finalice la ejecución del flujo de trabajo. En **Summary**, puedes ver las dos tareas del flujo de trabajo, el estado y los artefactos retenidos de la ejecución. Puedes hacer clic en cada trabajo para revisar los registros.

    ![Flujo de trabajo correcto](images/gh-action-success.png)

4. En una ventana independiente, vuelve a Azure Portal (https://portal.azure.com/). Abre el grupo de recursos creado antes. Verás que la acción de GitHub, mediante una plantilla de bicep, ha creado un plan de Azure App Service + App Service. Para ver el sitio web publicado, abre App Service y haz clic en **Examinar**.

    ![Examinar WebApp](images/browse-webapp.png)

#### (OPCIONAL) Tarea 4: Agregar la aprobación manual previa a la implementación mediante entornos de GitHub

En esta tarea, usarás entornos de GitHub para pedir la aprobación manual antes de ejecutar las acciones definidas en el trabajo de implementación del flujo de trabajo.

1. En la página del repositorio, ve a **Code** y abre el siguiente archivo: **eShopOnWeb/.github/workflows/eshoponweb-cicd.yml**.
2. En la sección **Implementar trabajo**, puedes encontrar una referencia a un **entorno** denominado **Desarrollo**. Los [entornos](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) usados de GitHub agregan reglas de protección (y secretos) para los destinos.

3. En la página del repositorio, ve a **Settings**, abre **Environments** y haz clic en **New environment**.
4. Asígnale el nombre **Desarrollo** y haz clic en **Configure Environment**.

    > NOTA: Si ya existe un entorno denominado **Desarrollo** en la lista **Environments**, haz clic en el nombre del entorno para abrir su configuración.  
    
5. En la pestaña **Configure Development**, marca la opción **Required Reviewers** y tu cuenta de GitHub como revisor. Haz clic en **Save protection rules**.
6. Ahora, vamos a probar la regla de protección. En la página del repositorio, ve a **Actions**, haz clic en el flujo de trabajo **eShopOnWeb Build and Test** y haz clic en **Run workflow>Run workflow** para la ejecución manual.

    ![flujo de trabajo de desencadenador manual](images/gh-manual-run.png)

7. Haz clic en la ejecución iniciada del flujo de trabajo y espera a que finalice el trabajo **buildandtest**. Verás una solicitud de revisión cuando llegues al trabajo **implementación**.

8. Haz clic en **Review deployments**, marca **Development** y haz clic en **Approve and deploy**.

    ![aprobación](images/gh-approve.png)

9. El flujo de trabajo seguirá la ejecución del trabajo de **implementación** y finalizará.

### Ejercicio 2: Quitar los recursos del laboratorio de Azure

En este ejercicio, usarás Azure Cloud Shell para quitar los recursos de Azure aprovisionados en este laboratorio para eliminar cargos innecesarios.

1. En Azure Portal, abra la sesión de shell de **Bash** en el panel **Cloud Shell**.
2. Ejecute el comando siguiente para enumerar todos los grupos de recursos que se han creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-eshoponweb')].name" --output tsv
    ```

3. Ejecute el comando siguiente para eliminar todos los grupos de recursos que ha creado en los laboratorios de este módulo:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-eshoponweb')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro --nowait). Aunque podrá ejecutar otro comando de la CLI de Azure inmediatamente después en la misma sesión de Bash, los grupos de recursos tardarán unos minutos en quitarse.

## Revisar

En este laboratorio, implementaste un flujo de trabajo de Acción de GitHub que implementa una aplicación web de Azure.