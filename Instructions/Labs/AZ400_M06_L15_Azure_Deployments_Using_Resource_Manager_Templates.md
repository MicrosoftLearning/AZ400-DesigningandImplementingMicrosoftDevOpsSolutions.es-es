---
lab:
  title: Implementaciones mediante plantillas de Azure Bicep
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# Implementaciones mediante plantillas de Azure Bicep

# Manual de laboratorio para alumnos

## Requisitos del laboratorio

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurar una organización de Azure DevOp:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identifique una suscripción de Azure existente o cree una.

- Compruebe que tenga una cuenta de Microsoft o Azure AD con el rol Propietario en la suscripción de Azure y el rol Administrador global en el inquilino de Azure AD asociado a la suscripción de Azure.

- [Visual Studio Code](https://code.visualstudio.com/). Esto se instalará como parte de los requisitos previos para este laboratorio.

## Introducción al laboratorio

En este laboratorio, crearás una plantilla de Azure Bicep y la modularizarás mediante el concepto de módulos de Azure Bicep. A continuación, modificarás la plantilla de implementación principal para usar el módulo y, por último, implementarás todos los recursos en Azure.

## Objetivos

Después de completar este laboratorio, podrá:

- Comprender y crear plantillas de Azure Bicep.
- Crear un módulo de Bicep reutilizable para los recursos de almacenamiento.
- Subir una plantilla vinculada para Azure Blob Storage y generar el token de SAS.
- Modificar la plantilla principal para usar el módulo.
- Modificar la plantilla principal para actualizar las dependencias.
- Implementar todos los recursos en Azure mediante plantillas de Azure Bicep.

## Tiempo estimado: 60 minutos

## Instrucciones

### Ejercicio 0: configuración de los requisitos previos del laboratorio

En este ejercicio, configurarás los requisitos previos para el laboratorio, que incluyen Visual Studio Code.

#### Tarea 1: Instalación y configuración de Git y Visual Studio Code

En esta tarea, instalarás Visual Studio Code. Si ya has implementado este requisito previo, puedes continuar directamente con la siguiente tarea.

1. Si aún no tienes instalado Visual Studio Code, desde el equipo del laboratorio, inicia un navegador web, navega hasta la [página de descarga de Visual Studio Code](https://code.visualstudio.com/) y descárgalo e instálalo.

### Ejercicio 1: Creación e implementación de plantillas de Azure Bicep.

En este laboratorio, crearás una plantilla de Azure Bicep y un módulo de plantilla. A continuación, modificarás la plantilla de implementación principal para usar el módulo de plantillas y actualizar las dependencias, y, por último, implementarás las plantillas en Azure.

#### Tarea 1: Creación de plantillas de Azure Bicep

En esta tarea, usará Visual Studio Code para crear una plantilla de Azure Bicep.

1. En el equipo de laboratorio, inicia Visual Studio Code, en Visual Studio Code, haz clic en el menú de nivel superior **Archivo**, en el menú desplegable, selecciona **Preferencias**, y en el menú en cascada, **Extensiones, en el cuadro de texto **Extensiones de búsqueda****, escribe **Bicep**, selecciona el publicado por Microsoft y haz clic en **Instalar** para instalar la compatibilidad con el lenguaje Azure Bicep.
1. En un explorador web, conéctate con **<https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows/main.bicep>**. Haz clic en la opción **Sin procesar** del archivo. Copia el contenido de la ventana de código y pégalo en el editor de Visual Studio Code.

   > **Nota**: En lugar de crear una plantilla desde cero, usaremos una de las [plantillas de inicio rápido de Azure ](https://azure.microsoft.com/en-us/resources/templates/) denominada **Implementación de una máquina virtual de plantilla de Windows sencilla**. Las plantillas se pueden descargar desde GitHub: [vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows).

1. En el equipo de laboratorio, abre Explorador de archivos y crea la siguiente carpeta local que servirá para almacenar plantillas:

   - **C:\\templates**

1. Vuelve a la ventana de Visual Studio Code con nuestra plantilla main.bicep, haz clic en el menú de nivel superior **Archivo**, en el menú desplegable, haz clic en **Guardar como** y guarda la plantilla como **main.bicep** en la carpeta local recién creada **C:\\templates**.
1. Revisa la plantilla para comprender mejor su estructura. Existen tipos cinco recursos incluidos en la plantilla:

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. En Visual Studio Code, vuelve a guardar el archivo, pero esta vez elige **C:\\templates** como destino y **storage.bicep** como nombre de archivo.

   > **Nota**: Ahora tenemos dos archivos JSON idénticos: **C:\\templates\\main.bicep** y **C:\\templates\\storage.bicep**.

#### Tarea 2: Crear una plantilla vinculada para los recursos de almacenamiento.

En esta tarea, modificarás las plantillas que guardó en la tarea anterior, de modo que el módulo de plantilla de almacenamiento **storage.bicep** crearás solo una cuenta de almacenamiento, la importará la primera plantilla. El módulo de plantilla de almacenamiento debe volver a pasar un valor a la plantilla principal, **main.bicep**, y este valor se definirá en el elemento outputs del módulo de plantilla de almacenamiento.

1. En el archivo **storage.bicep** que se muestra en la ventana de Visual Studio Code, en la **sección recursos**, elimina todos los elementos de recurso excepto el recurso **storageAccounts**. Debe dar como resultado una sección de recursos que se muestra de la siguiente manera:

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. A continuación, elimina todas las definiciones de variables:

   ```bicep
   var storageAccountName = 'bootdiags${uniqueString(resourceGroup().id)}'
   var nicName = 'myVMNic'
   var addressPrefix = '10.0.0.0/16'
   var subnetName = 'Subnet'
   var subnetPrefix = '10.0.0.0/24'
   var virtualNetworkName = 'MyVNET'
   var networkSecurityGroupName = 'default-NSG'
   var securityProfileJson = {
     uefiSettings: {
       secureBootEnabled: true
       vTpmEnabled: true
     }
     securityType: securityType
   }
   var extensionName = 'GuestAttestation'
   var extensionPublisher = 'Microsoft.Azure.Security.WindowsAttestation'
   var extensionVersion = '1.0'
   var maaTenantName = 'GuestAttestation'
   var maaEndpoint = substring('emptyString', 0, 0)
   ```

1. A continuación, elimina todos los valores de parámetro, excepto la ubicación, y agrega el código de parámetro siguiente, lo que da como resultado el siguiente:

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string
   ```

1. A continuación, al final del archivo, elimina la salida actual y añade una nueva denominada valor de salida storageURI. Modifica la salida para que tenga el siguiente aspecto.

   ```bicep
   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

1. Guarda el módulo de plantilla storage.bicep. La plantilla de almacenamiento debería tener ahora el siguiente aspecto:

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string

   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }

   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

#### Tarea 3: Modificar la plantilla principal para usar el módulo de plantilla

En esta tarea, modificarás la plantilla principal para hacer referencia al módulo de plantilla que creaste en la tarea anterior.

1. En Visual Studio Code, haz clic en el menú de nivel superior **Archivo**, en el menú desplegable, selecciona **Abrir archivo**, en el cuadro de diálogo Abrir archivo, ve a **C:\\templates\\main.bicep**, selecciónalo y haz clic en **Abrir**.
1. En el archivo **main.bicep**, en la sección de recursos, elimina el elemento de recurso de almacenamiento.

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. A continuación, agrega el código siguiente directamente en la misma ubicación donde estaba el elemento de recurso de almacenamiento recién eliminado:

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. También es necesario modificar la referencia al URI del blob de la cuenta de almacenamiento en nuestro recurso de máquina virtual para usar la salida del módulo en su lugar. Busca el recurso de máquina virtual y reemplaza la sección diagnosticsProfile por lo siguiente:

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. Revisa los siguientes detalles en la plantilla principal:

   - Un módulo de la plantilla principal se utiliza para enlazar con otra plantilla.
   - El módulo tiene un nombre simbólico denominado storageModule. Este nombre se usa para configurar cualquier dependencia.
   - Solo puedes usar el modo de implementación incremental cuando se utilizan módulos de plantilla.
   - Se usa una ruta de acceso relativa para el módulo de plantilla.
   - Usa parámetros para pasar valores de la plantilla principal a la plantilla vinculada.

> **Nota**: Con las plantillas de Azure ARM, habrías usado una cuenta de almacenamiento para cargar la plantilla vinculada para facilitar su uso a otros usuarios. Con los módulos de Azure Bicep, tienes la opción de cargarlos en el registro de módulos de Azure Bicep, que tiene opciones de registro público y privado. Puedes encontrar más información en la [documentación de Azure Automanage](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules#file-in-registry).

1. Guarde la plantilla.

#### Tarea 4: Implementación de recursos en Azure mediante módulos de plantilla

> **Nota**: Puedes implementar plantillas de varias maneras, como el uso de la CLI de Azure instalada en el entorno local, desde Azure Cloud Shell o desde una canalización de CI/CD. En este laboratorio, vas a utilizar Azure CLI desde el Azure Cloud Shell.

> **Nota**: A diferencia de las plantillas de ARM, no puedes usar Azure Portal para implementar directamente plantillas de Bicep.

> **Nota**: Para usar Azure Cloud Shell, cargarás los archivos main.bicep y storage.bicep en tu directorio principal de Cloud Shell.

> **Nota**: Actualmente, la CLI de Azure no admite la implementación de archivos Bicep remotos. Puedes compilar los archivos bicep para obtener el JSON de la plantilla de ARM y después cargarlos en una cuenta de almacenamiento, para luego implementarlos de forma remota.

1. En el equipo de laboratorio, en el explorador web que muestra Azure Portal, haz clic en el icono de **Cloud Shell** para abrir Cloud Shell.
   > **Nota**: Si todavía tienes la sesión de PowerShell anterior en este ejercicio activa, cambia a Bash (paso siguiente).
1. En el panel de Cloud Shell, haz clic en **PowerShell**, en el menú desplegable, haz clic en **Bash** y, cuando se te solicite, haz clic en **Confirmar**.
1. En el panel de Cloud Shell, haz clic en el icono **Cargar/Descargar archivos**, en el menú desplegable haz clic en **Cargar**.
1. En el cuadro de diálogo **Abrir**, ve a **C:\\templates\\main.bicep** y selecciónalo y haz clic en **Abrir**.
1. Sigue los mismos pasos para cargar también el archivo **storage.bicep** de C:\\templates\\.
1. Desde una sesión de **Bash** en el panel de Cloud Shell, ejecuta lo siguiente para realizar una implementación mediante una plantilla recién cargada:

   ```bash
   az deployment group what-if --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. Cuando se te solicite proporcionar el valor de "adminUsername", escribe **Student** y presiona la tecla **Enter**.
1. Cuando se te solicite proporcionar el valor de "adminPassword", escribe **Pa55w.rd1234** y presiona la tecla **Enter**. (No se mostrará la contraseña)
1. Revisa el resultado de este comando que valida la implementación y te informa si hay algún error en las plantillas. Esto tiene mucho valor, especialmente al implementar plantillas con muchos recursos y en entornos de nube importantes para la empresa.

1. Desde una sesión de **Bash** en el panel de Cloud Shell, ejecuta lo siguiente para realizar una implementación con una plantilla recién cargada:

   ```bash
   LOCATION='<region>'
   ```
   > **Nota**: Cambia el nombre de la región a una región cercana a tu ubicación. Si no sabes qué ubicaciones hay disponibles, ejecuta el comando `az account list-locations -o table`.
  
   ```bash
   az group create --name az400m06l15-RG --location $LOCATION
   ```

   ```bash   
   az deployment group create --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. Cuando se te pida que proporciones el valor de "adminUsername", escribe **Student** y presiona la tecla **Entrar**.
1. Cuando se te pida que proporciones el valor de "adminPassword", escribe **Pa55w.rd1234** y presiona la tecla **Entrar**. (No se mostrará la contraseña)

1. Si recibes errores al ejecutar el comando anterior para implementar la plantilla, prueba lo siguiente:

   - Si tienes varias suscripciones de Azure, asegúrate de que has indicado correctamente el contexto de suscripción en el que se implementa el grupo de recursos.
   - Asegúrate de que la plantilla vinculada sea accesible a través del URI especificado.

> **Nota**: Como paso siguiente, podrías modular las definiciones de los recursos que quedan en la plantilla de implementación principal, como las definiciones del recurso de red y de máquina virtual.

> **Nota**: Si no planeas usar los recursos implementados, debes eliminarlos para evitar cargos asociados. Para ello, basta con eliminar el grupo de recursos **az400m06l15-RG**.

### Ejercicio 2: eliminación de los recursos del laboratorio de Azure

En este ejercicio, quitarás los recursos de Azure aprovisionados en este laboratorio para eliminar cargos inesperados.

> **Nota**: No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

#### Tarea 1: eliminar los recursos del laboratorio de Azure

En esta tarea, usarás Azure Cloud Shell para quitar los recursos de Azure aprovisionados en este laboratorio con el propósito de eliminar cargos innecesarios.

1. En Azure Portal, abra la sesión de shell de **Bash** en el panel **Cloud Shell**.
1. Ejecute el comando siguiente para enumerar todos los grupos de recursos que se han creado en los laboratorios de este módulo:

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].name" --output tsv
   ```

1. Ejecute el comando siguiente para eliminar todos los grupos de recursos que ha creado en los laboratorios de este módulo:

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Nota**: El comando se ejecuta de forma asincrónica (según determina el parámetro --nowait). Aunque podrá ejecutar otro comando de la CLI de Azure inmediatamente después en la misma sesión de Bash, los grupos de recursos tardarán unos minutos en quitarse.

## Revisar

En este laboratorio, has aprendido a crear una plantilla de Azure Resource Manager, a modularizarla con una plantilla vinculada, a modificar la plantilla de implementación principal para llamar a la plantilla vinculada y a las dependencias actualizadas y, por último, a implementar las plantillas en Azure.
