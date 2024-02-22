---
lab:
  title: Validación del entorno del laboratorio
  module: 'Module 0: Welcome'
---

# Validación del entorno del laboratorio

## Manual de laboratorio para alumnos

## Instrucciones para crear una organización de Azure DevOps (solo tienes que hacerlo una vez)

### Comience aquí si no tiene una suscripción de Azure.

1. Obtén un nuevo **código de promoción de Azure Pass** del instructor u otro origen.
1. Usa una sesión privada del explorador para obtener una nueva **cuenta Microsoft (MSA) personal** en [https://account.microsoft.com](https://account.microsoft.com).
1. Con la misma sesión del explorador, ve a [https://www.microsoftazurepass.com](https://www.microsoftazurepass.com) para canjear tu pase de Azure mediante tu cuenta Microsoft (MSA). Para más información, consulta [Canje de un pase de Microsoft Azure](https://www.microsoftazurepass.com/Home/HowTo?Length=5). Sigue las instrucciones de canje

### Comience aquí si tiene una suscripción de Azure.

1. Abre un explorador y ve a [https://portal.azure.com](https://portal.azure.com), y luego busca en la parte superior de la pantalla de Azure Portal para **Azure DevOps**. En la página resultante, haz clic en **Organizaciones de Azure DevOps**.
1. Después haz clic en el enlace con la etiqueta **My Azure DevOps Organizations ** o ve directamente a [https://aex.dev.azure.com](https://aex.dev.azure.com).
1. En la página **Necesitamos más detalles**, selecciona **Continuar**.
1. En el cuadro desplegable de la izquierda, elige **Directorio predeterminado**, en lugar de “Cuenta Microsoft”.
1. Si se te solicita (*“Necesitamos algunos detalles más”),* proporciona tu nombre, dirección de correo electrónico y ubicación y haz clic en **Continuar**.
1. De nuevo en [https://aex.dev.azure.com](https://aex.dev.azure.com) con **el directorio predeterminado** seleccionado, haz clic en el botón azul **Crear nueva organización.**
1. Acepta los *Términos de servicio* haciendo clic en **Continuar**.
1. Si se te solicita (*“Casi hecho”),* deja el nombre de la organización de Azure DevOps de forma predeterminada (debe ser un nombre único global) y elige una ubicación de hospedaje cercana a ti en la lista.
1. Una vez que se abra la organización recién creada en **Azure DevOps**, haz clic en **Configuración de la organización ** en la esquina inferior izquierda.
1. En la pantalla **Configuración de la organización **, haz clic en **Facturación** (abrir esta pantalla tarda unos segundos).
1. Haz clic en **Configuración de facturación** y, en el lado derecho de la pantalla, selecciona la suscripción de**Pase de Azure - Patrocinio** y haz clic en **Guardar** para vincular la suscripción con la organización.
1. Una vez que la pantalla muestre el identificador de suscripción de Azure vinculado en la parte superior, cambia el número de **trabajos paralelos de pago** por **CI/CD** hospedados de MS de 0 a **1**. A continuación, haz clic en el botón **GUARDAR** en la parte inferior.
1. En **Organización Configuración**, ve a la sección **Canalizaciones** y haz clic en **Configuración**.
1. Pulsa el botón a**Desactivar** para **Deshabilitar la creación de canalizaciones de compilación clásicas** y **Deshabilitar la creación de canalizaciones de versión clásicas**.
    > Nota: El interruptor **Deshabilitar la creación de canalizaciones de versión clásicas** establece en **Activado** oculta las opciones de creación de canalizaciones de versión clásicas, como el menú **Versión** de la sección **Canalización** de proyectos de DevOps.
1. En **Organización Configuración**, ve a la sección **Seguridad** y haz clic en **Directivas**.
1. Cambia el interruptor a ** Activado ** para ** el acceso a aplicaciones de terceros a través de OAuth.**
    > Nota: La configuración de OAuth ayuda a habilitar herramientas como DemoDevOpsGenerator para registrar extensiones. Sin esto, puede producirse un error en varios laboratorios, debido a la falta de las extensiones necesarias.
1. Cambie el interruptor a **Activado** para **permitir proyectos públicos.**
    > Nota: Las extensiones usadas en algunos laboratorios pueden requerir un proyecto público para permitir el uso de la versión gratuita.
1. **Espera al menos 3 horas antes de usar las funcionalidades de CI/CD** para que la nueva configuración se refleje en el backend. De lo contrario, verás el mensaje *“No se ha comprado o concedido ningún paralelismo hospedado”.*

## Instrucciones para crear el proyecto de Azure DevOps de ejemplo (solo tienes que hacerlo una vez)

### Ejercicio 0: configuración de los requisitos previos del laboratorio

> **Nota**: Asegúrate de completar los pasos necesarios para crear la organización de Azure DevOps antes de continuar con estos pasos.

En este ejercicio, configurarás los requisitos previos para el laboratorio, lo que supone crear un nuevo proyecto de Azure DevOps con un repositorio basado en [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Tarea 1: Creación y configuración del proyecto de equipo

En esta tarea, crearás un proyecto de **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps. Haz clic en **Nuevo proyecto**. Asigna al proyecto la siguiente configuración:
    - nombre: **eShopOnWeb**
    - visibilidad: **Privado**
    - Avanzado: Control de versiones: **Git**
    - Avanzado: Proceso de elemento de trabajo: **Scrum**

1. Haga clic en **Crear**.

    ![Crear proyecto](images/create-project.png)

#### Tarea 2: importar el repositorio de Git de eShopOnWeb

En esta tarea, importarás el repositorio de Git eShopOnWeb que se usará en varios laboratorios.

1. En el equipo del laboratorio, en una ventana del explorador, abre la organización de Azure DevOps y el proyecto **eShopOnWeb** creado anteriormente. Haz clic en **Repos>Archivos**, **Importar un repositorio**. Seleccione **Import** (Importar). En la ventana **Importar un repositorio de Git**, pega la siguiente dirección URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> y haz clic en **Importar**:

    ![Importar repositorio](images/import-repo.png)

1. El repositorio se organiza de la siguiente manera:
    - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
    - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
    - La carpeta **infra** contiene la infraestructura de Bicep y ARM como plantillas de código usadas en algunos escenarios de laboratorio.
    - Definiciones de flujo de trabajo de GitHub del contenedor de carpetas **.github**.
    - La carpeta **src** contiene el sitio web de .NET 8 que se usa en los escenarios de laboratorio.

Ahora has completado los pasos previos necesarios para continuar con los diferentes laboratorios individuales para este curso AZ-400.
