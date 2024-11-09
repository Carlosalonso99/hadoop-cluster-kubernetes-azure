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

  - Si utilizas una **entidad de servicio** y no tienes suficientes permisos, podrías recibir una advertencia. En este caso:
    1. Inicia sesión con una cuenta de usuario.
    2. Ejecuta el siguiente comando para capturar el `oid` de la ubicación personalizada:
       ```bash
       az ad sp show --id bc313c14-388c-4e7d-a58e-70017303ee3b --query id -o tsv
       ```
    3. Inicia sesión nuevamente con la entidad de servicio y habilita las características usando este `oid`:
       ```bash
       az connectedk8s enable-features -n <cluster-name> -g <resource-group-name> --custom-locations-oid <cl-oid> --features cluster-connect custom-locations
       ```

### 4. Crear una Ubicación Personalizada

Las ubicaciones personalizadas permiten desplegar servicios de Azure directamente en el clúster Kubernetes. Este paso también lo realizaremos desde la interfaz gráfica de Azure.

## Paso 1: Crear un Clúster de Kubernetes (AKS) en Azure

### 1. Crear el Clúster en el Portal de Azure

1. Inicia sesión en el [Portal de Azure](https://portal.azure.com/).
2. En la barra de búsqueda, busca "Kubernetes Service" y selecciona **Azure Kubernetes Service (AKS)**.
3. Haz clic en **Crear** para crear un nuevo clúster.
   - **Grupo de Recursos**: Crea uno nuevo o usa uno existente.
   - **Nombre del Clúster**: Dale un nombre adecuado, como `hadoop-aks-cluster`.
   - **Región**: Selecciona la región más cercana o adecuada para tu caso.
   - **Nodo**: Elige un tamaño de nodo pequeño (1 o 2 nodos) para este ejercicio.

4. Revisión y Creación: Haz clic en **Revisar + crear** y luego en **Crear**. Este proceso tomará unos minutos.

### 2. Obtener las Credenciales del Clúster

Cuando el clúster esté listo, necesitarás obtener las credenciales para gestionarlo con `kubectl` desde la consola.

- **Obtener Credenciales del Clúster**:
  ```bash
  az aks get-credentials --resource-group <tu-grupo-de-recursos> --name <nombre-del-cluster>
  ```

Esto copiará las credenciales necesarias para gestionar el clúster desde `kubectl`.
