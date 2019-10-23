# Consolidating containerized apps with Azure Kubernetes Service

## Contenido
<!-- TOC -->
- [Consolidating containerized apps with Azure Kubernetes Service](#consolidating-containerized-apps-with-azure-kubernetes-service)
  - [Contenido](#contenido)
  - [Introducción](#introducci%c3%b3n)
  - [Pre requisitos](#pre-requisitos)
    - [Tarea 1: Iniciar sesión en Azure](#tarea-1-iniciar-sesi%c3%b3n-en-azure)
    - [Tarea 2: Definir las variables necesarias](#tarea-2-definir-las-variables-necesarias)
    - [Tarea 3: Desplegar los recursos necesarios](#tarea-3-desplegar-los-recursos-necesarios)
  - [Despliegue de AKS](#despliegue-de-aks)
  - [Tarea 1: Acceder a las versiones disponibles](#tarea-1-acceder-a-las-versiones-disponibles)
    - [Tarea 2: Despliegue del cluster de AKS](#tarea-2-despliegue-del-cluster-de-aks)
    - [Tarea 3: Configuación de Kubectl](#tarea-3-configuaci%c3%b3n-de-kubectl)
<!-- /TOC -->


## Introducción


## Pre requisitos

En caso de no tener una suscripción a Azure, se deberá iniciar sesion en el siguiente enlace para acceder a un trial gratis: [Create your Azure free account today](https://azure.microsoft.com/en-us/free/)

### Tarea 1: Iniciar sesión en Azure

1.  Iniciar sesión en: <https://portal.azure.com>.
2.  Abrir cloud Shell.

### Tarea 2: Definir las variables necesarias

```
BASE=AKSvOpen
PRESENTER='Victor'
LOCATION=eastus
LOCATION2=westus2
SUB='vOpen Workshop'
RG='WorkshopAKS'
RG2='WorkshopAKS2'
ACR_NAME='vOpenACR'
AKV_NAME=$BASE-$PRESENTER-vlt
DB_BASE=tailwind
PG_HOST_BASE=".postgres.database.azure.com"
PG_USER_BASE=tuser
PG_PASS='asdf1234)(*&^)'
COLLECTION=inventory
KUBERNETESVERSION=1.14.6
CLUSTER_NAME=AKSCluster
NODE_COUNT=3
```


### Tarea 3: Desplegar los recursos necesarios

1. Seleccionar la suscripción que vamos a utilizar para desplegar los recursos:
```
az account set --subscription ""
```
2. Verificar que la suscripción sea la correcta, revisando los grupos de recursos desplegados:
```
az group list -o table
```
3. Crear los grupos de recursos:
```
az group create --resource-group $RG --location $LOCATION
az group create --resource-group $RG2 --location $LOCATION2
```
4. Desplegar un Azure Container Registry
```
az acr create --resource-group $RG --name $ACR_NAME --sku Standard --location $LOCATION
az acr update -n $ACR_NAME --admin-enabled true
```
5. Definir el lugar donde se alojará la configuración de Kubernetes:
```
export KUBECONFIG=~/.kube/config
```
6. Desplegar un Azure Key Vault
```
az keyvault create --resource-group $RG --name $AKV_NAME
```

## Despliegue de AKS

## Tarea 1: Acceder a las versiones disponibles


1.Listar todas las versiones disponibles:
```
az aks get-versions -l $LOCATION -o table
```
2. Listar la última versión de Kubernetes:
```
az aks get-versions -l $LOCATION --query 'orchestrators[-1].orchestratorVersion' -o tsv

```


### Tarea 2: Despliegue del cluster de AKS

```
az aks create --resource-group $RG --name $CLUSTER_NAME --node-count $NODE_COUNT --kubernetes-version $KUBERNETESVERSION \
--enable-addons monitoring,http_application_routing --generate-ssh-keys
```

```
CLIENT_ID=$(az aks show --resource-group $RG --name $CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv)
```
```
# Get the ACR registry resource id
ACR_ID=$(az acr show --name $ACR_NAME --resource-group $RG --query "id" --output tsv)
```

```
# Create role assignment
az role assignment create --assignee $CLIENT_ID --role Reader --scope $ACR_ID
```

```
# Show http application routing zone
az aks show --resource-group $RG --name $CLUSTER_NAME --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName -o tsv
```

### Tarea 3: Configuación de Kubectl
1. Configurar credenciales:
```
az aks get-credentials --resource-group $RG --name $CLUSTER_NAME --file ~/.kube/config --overwrite-existing
```
2. Obtener nodos desde Kubectl:
```
kubectl get nodes
```
