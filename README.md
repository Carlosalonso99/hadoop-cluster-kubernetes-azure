# Despliegue de un Clúster de Hadoop en Azure con Docker y Kubernetes

Este README proporciona una guía paso a paso para desplegar un clúster de Hadoop en Azure utilizando Docker y Kubernetes. Utilizaremos Azure Kubernetes Service (AKS), Docker, y algunas funcionalidades adicionales como Azure Arc y ubicaciones personalizadas para optimizar la gestión del clúster.

## Requisitos Previos

### 1. Instalar y Actualizar Azure CLI y Extensiones

- **Instalar Azure CLI**: Si aún no tienes instalada Azure CLI, puedes instalarla con el siguiente comando:
  ```bash
  curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
  ```

- **Instalar extensiones necesarias**: Asegúrate de tener las versiones más recientes de las siguientes extensiones:
  - `connectedk8s`
  - `k8s-extension`
  - `customlocation`

  Ejecuta los siguientes comandos para instalarlas:
  ```bash
  az extension add --name connectedk8s --allow-preview true
  az extension add --name k8s-extension --allow-preview true
  az extension add --name customlocation --allow-preview true
  ```

  Si ya tienes las extensiones instaladas, actualízalas:
  ```bash
  az extension update --name connectedk8s --allow-preview true
  az extension update --name k8s-extension --allow-preview true
  az extension update --name customlocation --allow-preview true
  ```

### 2. Registrar el Proveedor `Microsoft.ExtendedLocation`

El proveedor `Microsoft.ExtendedLocation` es necesario para usar Azure Arc y funcionalidades relacionadas con ubicaciones personalizadas.

- **Registrar el proveedor**:
  ```bash
  az provider register --namespace Microsoft.ExtendedLocation
  ```

- **Comprobar el estado del registro**:
  ```bash
  az provider show -n Microsoft.ExtendedLocation -o table
  ```
  Espera a que el estado sea `Registered`, lo cual puede tardar unos minutos.

### 3. Habilitar Azure Arc en el Clúster AKS

Azure Arc te permite gestionar clústeres de Kubernetes fuera de Azure, como si fueran recursos nativos de Azure. Este paso lo realizaremos desde la interfaz gráfica de Azure.

### 4. Crear una Ubicación Personalizada

Las ubicaciones personalizadas permiten desplegar servicios de Azure directamente en el clúster Kubernetes. Este paso también lo realizaremos desde la interfaz gráfica de Azure.

## Paso 1: Crear un Clúster de Kubernetes (AKS) en Azure

### 1. Crear el Clúster en el Portal de Azure

1. Inicia sesión en el [Portal de Azure](https://portal.azure.com/).
2. En la barra de búsqueda, busca "Kubernetes Service" y selecciona **Azure Kubernetes Service (AKS)**.
3. Haz clic en **Crear** para crear un nuevo clúster.
   - **Grupo de Recursos**: Selecciona `hadoop-kubernetes` que ya has creado.
   - **Nombre del Clúster**: Introduce `hadoop-cluster`.
   - **Región**: Selecciona la región que sea más adecuada para ti (por ejemplo, `West Europe` o la más cercana a tu ubicación).
   - **Nodo**: Elige un tamaño de nodo pequeño (1 o 2 nodos) para este ejercicio.

4. Revisión y Creación: Haz clic en **Revisar + crear** y luego en **Crear**. Este proceso tomará unos minutos.

### 2. Conectar el Clúster a Azure Arc

1. **Navegar a Azure Arc**:
   - En la barra de búsqueda del portal, busca **Azure Arc** y selecciona **Azure Arc**.

2. **Añadir Clúster de Kubernetes a Azure Arc**:
   - Dentro de Azure Arc, selecciona **Clústeres de Kubernetes**.
   - Haz clic en **Agregar** y selecciona **Conectar un clúster de Kubernetes existente**.

3. **Seleccionar el Método de Conexión**:
   - Selecciona **Método de conexión manual**.
   - Haz clic en **Siguiente: Autenticación**.

4. **Autenticación y Scripts**:
   - Azure te mostrará un script que debes ejecutar para conectar el clúster. Este script es necesario para registrar el clúster con Azure Arc.
   - Copia el script y ejecútalo en el terminal de la máquina que tenga acceso al clúster AKS.
   - El script realizará la conexión del clúster a Azure Arc y lo registrará en el portal.

5. **Verificar la Conexión en el Portal**:
   - Una vez que el script haya finalizado con éxito, regresa al portal y verifica si el clúster aparece bajo **Clústeres de Kubernetes** en Azure Arc.

### 3. Habilitar Ubicaciones Personalizadas y Crear la Ubicación

1. **Navegar a Azure Arc y Habilitar Características**:
   - Selecciona el clúster `hadoop-cluster` que aparece en Azure Arc.
   - Dentro del clúster, selecciona la pestaña **Características** o **Features**.
   - Busca y habilita las características `cluster-connect` y `custom-locations`.

2. **Crear una Ubicación Personalizada**:
   - En el portal de Azure, ve al recurso **Azure Arc** y selecciona **Ubicaciones personalizadas**.
   - Haz clic en **Crear**.
   - **Nombre de la Ubicación**: Escribe `hadoop-custom-location` o un nombre que prefieras.
   - **Grupo de Recursos**: Selecciona `hadoop-kubernetes`.
   - **Espacio de Nombres (Namespace)**: Puedes usar el espacio de nombres predeterminado `default`.
   - **Clúster de Kubernetes Host**: Selecciona el clúster `hadoop-cluster` que has conectado previamente.

3. **Crear la Ubicación**:
   - Haz clic en **Revisar + crear** y luego en **Crear**.

### 4. Verificar la Ubicación Personalizada

1. **Verificar la Ubicación**:
   - Vuelve al grupo de recursos `hadoop-kubernetes` y verifica que la ubicación personalizada `hadoop-custom-location` esté creada y asociada correctamente.

2. **Utilizar la Ubicación Personalizada**:
   - Ahora deberías poder seleccionar la ubicación personalizada en otras configuraciones de Azure, como la creación de servicios o recursos adicionales que quieres asociar con el clúster.

## Paso 2: Obtener las Credenciales del Clúster

Cuando el clúster esté listo, necesitarás obtener las credenciales para gestionarlo con `kubectl` desde la consola.

- **Obtener Credenciales del Clúster**:
  ```bash
  az aks get-credentials --resource-group hadoop-kubernetes --name hadoop-cluster
  ```

Esto copiará las credenciales necesarias para gestionar el clúster desde `kubectl`.
