---
lab:
  title: Uso compartido del conocimiento de equipo con las wikis de proyectos de Azure
  module: 'Module 09: Implement continuous feedback'
---

# Uso compartido del conocimiento de equipo con las wikis de proyectos de Azure

## Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/azure/devops/server/compatibility).

- **Configurar una organización de Azure DevOps:**: si aún no tienes una organización Azure DevOps que puedas usar para este laboratorio, crea una siguiendo las instrucciones disponibles en [Requisitos previos para el laboratorio AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Configure el proyecto eShopOnWeb de ejemplo:** Si aún no tiene el proyecto eShopOnWeb de ejemplo que puede usar para este laboratorio, cree uno siguiendo las instrucciones disponibles en [Requisitos previos de laboratorio de AZ-400](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

## Introducción al laboratorio

En este laboratorio, creará y configurará una wiki en Azure DevOps, incluida la administración del contenido de Markdown y la creación de un diagrama de Mermaid.

## Objetivos

Después de completar este laboratorio, podrá:

- Cree una wiki en un proyecto de Azure.
- Agregue y edite el contenido de Markdown.
- Cree un diagrama de Mermaid.

## Tiempo estimado: 45 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, queremos recordarle debe validar los prerrequisitos del laboratorio, teniendo tanto una organización de Azure DevOps lista, como haber creado el proyecto eShopOnWeb. Consulta las instrucciones para obtener más detalles.

### Ejercicio 1: publicación de código como wiki

En este ejercicio, aprenderás el proceso de publicación de un repositorio de Azure DevOps como wiki y de administración de la wiki publicada.

> **Nota**: el contenido que mantienes en un repositorio de Git se puede publicar en una wiki de Azure DevOps. Por ejemplo, el contenido escrito para admitir un kit de desarrollo de software, documentación del producto o archivos LÉAME se puede publicar directamente en una wiki. Tienes la opción de publicar varias wikis en el mismo proyecto de equipo de Azure DevOps.

#### Tarea 1: publicar una rama de un repositorio de Azure DevOps como wiki

En esta tarea, publicarás una rama de un repositorio de Azure DevOps como wiki.

> **Nota**: si la wiki publicada corresponde a una versión del producto, puedes publicar nuevas ramas a medida que publicas nuevas versiones del producto.

1. En el menú vertical del lado izquierdo, haga clic en **Repositorios**, en la sección superior del panel **Archivos**, asegúrese de que tiene seleccionado el repositorio **eShopOnWeb** (selecciónelo en la lista desplegable de la parte superior con el icono de Git). En la lista desplegable de la rama (arriba de "Archivos" con el icono de rama), selecciona **main** y revisa el contenido de la rama principal.
1. A la izquierda del panel **Archivos**, en la lista de la carpeta del repo y en la jerarquía de archivos, expande la carpeta **src** y ve a la subcarpeta **Web-> wwwroot -> imágenes**. En la subcarpeta **Imágenes**, busca la entrada **brand.png**, pasa el puntero del mouse sobre el extremo derecho para ver los puntos suspensivos verticales (tres puntos) que representa el menú **Más**, haz clic en **Descargar** para descargar el archivo **brand.png** en la carpeta local **Descargas** del equipo del laboratorio.

    >**Nota**: usarás esta imagen en el siguiente ejercicio de este laboratorio.

1. Almacenaremos los archivos de origen de la wiki en una carpeta independiente dentro de la estructura de carpetas actual de Repos. Desde **Repos**, selecciona **Archivos**. Observe el título del repositorio **eShopOnWeb** en la parte superior de la estructura de carpetas. **Selecciona los puntos suspensivos (3 puntos)**, elige **Nuevo/Carpeta** y asígnale el nombre **Documentos** a la carpeta nueva. Como un repo no permite crear una carpeta vacía, asígnale el nombre **READ.ME** al archivo nuevo.
1. Confirma la creación de la carpeta y el archivo presionando **pulsando el botón Crear**.
1. El archivo READ.ME se abrirá en el modo de vista integrada. Dado que se almacena "como código", debes **Confirmar** los cambios haciendo clic en el botón **Confirmar**. En la ventana Confirmar, presiona **Confirmar** una vez más.
1. En el menú vertical de Azure DevOps del lado izquierdo, haz clic en **Información general**; en la sección **Información general**, selecciona **Wiki** y **Publicar código como wiki*.
1. En el panel **Publicar código como wiki**, especifica la siguiente configuración y haz clic en **Publicar**.

    | Configuración | Valor |
    | ------- | ----- |
    | Repositorio | **eShopOnWeb** |
    | Sucursal | **principal** |
    | Carpeta | **Documentos** |
    | Nombre de wiki | **eShopOnWeb (documentos)** |

    >**Nota**: esto abrirá automáticamente la sección Wiki y publicará **el editor**, donde puedes introducir un título de página wiki y agregar contenido. Ten en cuenta que se recomienda usar el formato MarkDown, pero usa la cinta de opciones para ayudar con algunas de las sintaxis de diseño de MarkDown.

1. En el campo **Título** de la página wiki, escribe "Te damos la bienvenida a nuestra tienda en línea".

1. En el cuerpo de la página wiki, pega el texto siguiente:

    ```text
    ##Welcome to Our Online Retail Store!
    At our online retail store, we offer a **wide range of products** to meet the **needs of our customers**. Our selection includes everything from *clothing and accessories to electronics, home decor, and more*.
    
    We pride ourselves on providing a seamless shopping experience for our customers. Our website offers the following benefits:
    1. user-friendly,
    1. and easy to navigate, 
    1. allowing you to find what you're looking for,
    1. quickly and easily. 
    
    We also offer a range of **_payment and shipping options_** to make your shopping experience as convenient as possible.
    
    ### about the team
    Our team is dedicated to providing exceptional customer service. If you have any questions or concerns, our knowledgeable and friendly support team is always available to assist you. We also offer a hassle-free return policy, so if you're not completely satisfied with your purchase, you can easily return it for a refund or exchange.
    
    ### Physical Stores
    |Location|Area|Hours|
    |--|--|--|
    | New Orleans | Home and DIY  |07.30am-09.30pm  |
    | Seattle | Gardening | 10.00am-08.30pm  |
    | New York | Furniture Specialists  | 10.00am-09.00pm |
    
    ## Our Store Qualities
    - We're committed to providing high-quality products
    - Our products are offered at affordable prices 
    - We work with reputable suppliers and manufacturers 
    - We ensure that our products meet our strict standards for quality and durability. 
    - Plus, we regularly offer sales and discounts to help you save even more.
    
    #Summary
    Thank you for choosing our online retail store for your shopping needs. We look forward to serving you!
    ```

1. Este texto de ejemplo proporciona información general sobre varias de las características comunes de sintaxis markDown, como título y subtítulos (##), negrita (**), cursiva (*), cómo crear tablas, etc.

1. Al finalizar **presiona** el botón Guardar en la esquina superior derecha.

1. **Actualiza** el explorador o selecciona cualquier otra opción del portal de DevOps, y vuelve a la sección Wiki. Ahora verás la Wiki de **EshopOnWeb (documentos)** y la página **Te damos la bienvenida a nuestra tienda en línea** como **página principal** de la Wiki.

#### Tarea 2: administrar el contenido de una wiki publicada

En esta tarea, administrarás el contenido de la wiki que publicaste en la tarea anterior.

1. En el menú vertical del lado izquierdo, haga clic en **Repositorios**, asegúrese de que el menú desplegable de la sección superior del panel **Archivos** muestra el repositorio **eShopOnWeb** y la rama **principal**, en la jerarquía de carpetas del repositorio, seleccione la carpeta **Documentos** y seleccione el archivo **Welcome-to-our-Online-Retail-Store!.md**.
1. Verás cómo el formato MarkDown está visible aquí como texto sin formato, lo que te permite seguir editando el contenido del archivo desde aquí también.

> **Nota**: dado que los archivos de código fuente wiki se controlan como código fuente, recuerda que todas las prácticas del control de código fuente tradicional (clonar, solicitudes de incorporación de cambios, Aprobaciones y más) ahora también se pueden aplicar a las páginas wiki.

### Ejercicio 2: creación y administración de una wiki de proyecto

En este ejercicio, aprenderás a crear y administrar una wiki de proyecto.

> **Nota**: puedes crear y administrar una wiki independientemente de los repositorios existentes.

#### Tarea 1: crear una wiki de proyecto que incluye un diagrama de Mermaid y una imagen

En esta tarea, crearás una wiki de proyecto en la que agregarás un diagrama de Mermaid y una imagen.

1. En el equipo de laboratorio, en el portal de Azure DevOps que muestra el **panel Wiki** del proyecto **EShopOnweb**, con el contenido de la wiki **eShopOnWeb (documentos)** seleccionado, en la parte superior del panel, haga clic en el encabezado de la lista desplegable **eShopOnWeb (documentos)** (el icono de la flecha hacia abajo) y, en la lista desplegable, seleccione **Crear nueva wiki de proyecto**.
1. En el cuadro de texto **Título de la página**, escribe **Diseño del proyecto**.
1. Coloca el cursor en el cuerpo de la página, haz clic en el primer icono desde la izquierda de la barra de herramientas que representa la configuración del encabezado y, en la lista desplegable, haz clic en **Encabezado 1**. Esto agregará automáticamente el carácter hash (**#**) al principio de la línea.
1. Directamente después del carácter **#** recién agregado, escribe **Autenticación y autorización** y pulsa la tecla **Entrar.**
1. Haz clic en el primer icono desde la izquierda de la barra de herramientas que representa la configuración de encabezado y, en la lista desplegable, haz clic en **Encabezado 2**. Esto agregará automáticamente el carácter hash (**##**) al principio de la línea.
1. Directamente después del carácter **##** recién agregado, escribe **Flujo de autorización de OAuth 2.0 de Azure DevOps** y pulsa la tecla **Entrar.**
1. **Copia y pega** el código siguiente para insertar un diagrama de Mermaid en la wiki.

    ```text
    ::: mermaid
    sequenceDiagram
     participant U as User
     participant A as Your app
     participant D as Azure DevOps
     U->>A: Use your app
     A->>D: Request authorization for user
     D-->>U: Request authorization
     U->>D: Grant authorization
     D-->>A: Send authorization code
     A->>D: Get access token
     D-->>A: Send access token
     A->>D: Call REST API with access token
     D-->>A: Respond to REST API
     A-->>U: Relay REST API response
    :::
    ```

    >**Nota**: para obtener más información sobre la sintaxis de Mermaid, consulta [Acerca de Mermaid.](https://mermaid-js.github.io/mermaid/#/)

1. A la derecha del panel del editor, en el panel de vista previa, haz clic en **Cargar diagrama** y revisa el resultado.

    >**Nota**: el resultado debe ser similar al diagrama de flujo que muestra cómo [autorizar el acceso a las API de REST con OAuth 2.0](https://docs.microsoft.com/azure/devops/integrate/get-started/authentication/oauth)

1. En la esquina superior derecha del panel del editor, haz clic en el cursor de inserción hacia abajo junto al botón **Guardar** y, en el menú desplegable, haz clic en **Guardar con mensaje de revisión**.
1. En el cuadro de diálogo **Guardar página**, escribe **Sección de autenticación y autorización con el diagrama de Mermaid de OAuth 2.0** y haz clic en **Guardar**.
1. En el panel de editor **Diseño del proyecto**, coloca el cursor al final del elemento Mermaid que agregaste anteriormente, pulsa la tecla **Entrar** para agregar una línea adicional, haz clic en el icono izquierdo de la barra de herramientas que representa la configuración de encabezado y, en la lista desplegable, haz clic en **Encabezado 2**. Esto agregará automáticamente el carácter hash doble (**##**) al principio de la línea.
1. Directamente después del carácter **##** recién agregado, escribe **Interfaz de usuario** y pulsa la tecla **Entrar.**
1. En el panel del editor **Diseño del proyecto**, en la barra de herramientas, haz clic en el icono de clip que representa la acción **Insertar un archivo**, en el cuadro de diálogo **Abrir**, ve a la carpeta **Descargas**, selecciona el archivo **Brand.png** que descargaste en el ejercicio anterior y haz clic en **Abrir**.
1. Cuando vuelvas al panel del editor **Diseño del proyecto**, revisa el panel de vista previa y comprueba que la imagen se ve correctamente.
1. En la esquina superior derecha del panel del editor, haz clic en el cursor de inserción hacia abajo junto al botón **Guardar** y, en el menú desplegable, haz clic en **Guardar con mensaje de revisión**.
1. En el cuadro de diálogo **Guardar página**, escribe **Sección de la interfaz de usuario con la imagen de Tailwind Traders** y haz clic en **Guardar**.
1. Cuando vuelvas al panel del editor, haz clic en **Cerrar** en la esquina superior derecha.

#### Tarea 2: administrar una wiki de proyecto

En esta tarea, administrarás la wiki de un proyecto recién creado.

>**Nota**: para empezar, revertirás el cambio más reciente a la página wiki.

1. En el equipo de laboratorio, en el portal de Azure DevOps, en el que se muestra el **panel Wiki** del proyecto **eShopOnWeb**, con el contenido de la wiki **Diseño de proyecto** seleccionado, en la esquina superior derecha, haga clic en el símbolo de puntos suspensivos verticales y, en el menú desplegable, haga clic en **Ver revisiones**.
1. En el panel **Revisiones**, haz clic en la entrada que representa el cambio más reciente.
1. En el panel que aparece, revisa la comparación entre la versión anterior y la versión actual del documento, haz clic en **Revertir** y, cuando se solicite la confirmación, haz clic en **Revertir** de nuevo y haz clic en **Examinar página**.
1. Cuando vuelvas al panel **Diseño del proyecto**, comprueba si el cambio se revierte correctamente.

    >**Nota**: ahora agregarás otra página a la wiki del proyecto y la establecerás como la página principal de la wiki.

1. En el panel **Diseño del proyecto**, en la esquina inferior izquierda, haz clic en **+ Nueva página**.
1. En el panel del editor de páginas, en el cuadro de texto **Título de página**, escribe **Información general sobre el diseño del proyecto**, haz clic en **Guardar** y luego en **Cerrar**.
1. De nuevo en el panel donde se enumeran las páginas de la wiki del **proyecto del diseño**, busca la entrada **Información general sobre el diseño del proyecto**, selecciónala con el puntero del mouse, arrástrala y colócala sobre la entrada de la página **Diseño del proyecto**.
1. Confirma los cambios pulsando el botón **Mover** en la ventana que aparece.
1. Comprueba que la entrada **Información general sobre el diseño del proyecto** aparezca como la página de nivel superior con el icono de inicio que la identifica como página principal de la wiki.

## Revisar

En este laboratorio, crearás y configurarás una wiki en Azure DevOps, incluida la administración del contenido de Markdown y la creación de un diagrama de Mermaid.
