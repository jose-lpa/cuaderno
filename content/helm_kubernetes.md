Title: Gestor de paquetes Helm para Kubernetes
Date: 2020-04-07 10:48
Category: Kubernetes
Tags: cluster,kubernetes,devops,containers,helm
Slug: helm-kubernetes
Authors: José L. Patiño-Andrés
Summary: Configuración y uso básico del gestor de paquetes Helm para Kubernetes.

## Instalación

En la página de Helm podemos encontrar [la documentación](https://helm.sh/docs).

Para instalar Helm de forma manual podemos descargar los binarios de GitHub en
su [página de releases](https://github.com/helm/helm/releases).

También podemos instalarlo con Snap:

    :::bash
    sudo snap install helm

## Inicialización

Antes de usar Helm tenemos que inicializarlo:

    :::bash
    helm init

Quizá tengamos que añadir un nuevo repositorio desde el que descargar paquetes:

    :::bash
    helm repo add stable https://kubernetes-charts.storage.googleapis.com/

Es posible que la instalación de Helm no configure los permisos necesarios para
instalar paquetes efectivamente. En este caso tendremos que crear un *cluster role
binding*:

    :::bash
    kubectl create clusterrolebinding add-on-cluster-admin \
            --clusterrole=cluster-admin \
            --serviceaccount=kube-system:default

## Uso básico

Helm funciona en base a especificaciones de despliegue llamadas **Chart**. Un
*chart* es el equivalente a un paquete de software, una aplicación. Por
ejemplo podemos desplegar un sistema de CI Jenkins distribuido en nuestro
clúster Kubernetes usando el *chart* oficial `stable/jenkins`.

A continuación se añaden un par de comandos para un uso básico de Helm:

- `helm search`: muestra todos los *charts* disponibles para desplegar.
- `helm search <paquete>`: busca un *chart* determinado en base al texto
  introducido en `<paquete>`. Por ejemplo, si queremos buscar un servidor CI
  Jenkins:

```bash
user@machine:~$ helm search jenkins
NAME          	CHART VERSION	APP VERSION	DESCRIPTION
stable/jenkins	1.11.3       	lts        	Open source continuous integration server. It supports mu...
```

- `helm install <chart>`: instala el *chart* especificado. Por ejemplo, si
  queremos desplegar el servidor CI Jenkins en nuestro clúster k8s:

```bash
helm install stable/jenkins
```

- `helm list`: muestra los *charts* desplegados en nuestro clúster k8s. Esto
  mostrará en pantalla una salida tal que así:

```bash
user@machine:~$ helm list
NAME     	REVISION	UPDATED                 	STATUS  	CHART         	APP VERSION	NAMESPACE
hardy-olm	1       	Tue Apr  7 10:23:56 2020	DEPLOYED	jenkins-1.11.3	lts        	default
```

- `helm delete <NAME>`: borra todo lo relacionado con el *chart* al que se
  refiere `<NAME>`. Por ejemplo para eliminar del clúster nuestro sistema CI
  Jenkins anteriormente desplegado con el chart `jenkins-1.11.3`, haremos:

```bash
helm delete hardy-olm
```
