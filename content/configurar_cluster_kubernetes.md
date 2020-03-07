Title: Configurar cluster Kubernetes
Date: 2020-03-05 14:32
Category: Kubernetes
Tags: cluster,kubernetes,devops,containers
Slug: configurar-cluster-kubernetes
Authors: José L. Patiño-Andrés
Summary: Creación y configuración de un cluster Kubernetes con `kubeadm`.

### Intro

Este artículo es una muy pequeña referencia para configurar un cluster
Kubernetes. El cluster constará de un nodo "master" y un par de nodos más sobre
los que se podrán desplegar pods de aplicaciones de forma distribuida.

Para configurar nuestro cluster Kubernetes podemos hacer uso de las máquinas
virtuales creadas en el artículo ["Clúster de VMs con VirtualBox"]({filename}/virtualbox_cluster.md).

### Requisitos

Este cluster Kubernetes utilizará Docker como tecnología de contenedores. Por
tanto, tendremos que asegurarnos que las máquinas tienen instalado y configurado
Docker.

Hay que asegurarse también de que las máquinas del cluster tienen deshabilitada
la memoria de intercambio. Kubernetes no funcionará en máquinas que tengan
memoria de intercambio activa. Para deshabilitarla podemos ejecutar el siguiente
comando: `sudo swapoff -a` (funciona en Ubuntu 18.04).

Hay que instalar las aplicaciones `kubelet`, `kubeadm` y `kubectl`. Para ello
nos referiremos a la documentación oficial de Kubernetes, donde están las
instrucciones para instalar estas aplicaciones base para diferente sistemas
desde repositorios oficiales.

- `kubeadm`: es la aplicación que se usa para configurar y arrancar el cluster.
- `kubelet`: es el componente que se ejecuta en todas las máquinas y que se
  encarga de gestionar los nodos, iniciar pods y contenedores, etc.
- `kubectl`: es la aplicación CLI que utilizaremos para interactuar con el
  cluster.

### Inicialización del cluster

Para inicializar el cluster ejecutamos el siguiente comando con `kubeadm`:

    :::bash
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.56.2

La opción `--pod-network-cidr=10.244.0.0/16` la usamos cuando seleccionamos una
red de Pods [Flannel](https://github.com/coreos/flannel) para nuestro cluster
Kubernetes, lo cual se da por ejemplo en nuestro [cluster de VMs VirtualBox]({filename}/virtualbox_cluster.md).
Para más información sobre selección de una red de Pods (necesario), consultar
la documentación de Kubernetes.

La opción `--apiserver-advertise-address=192.168.56.2` la usamos para definir la
dirección IP del nodo donde se encuentra el servidor API de este cluster 
Kubernetes, es decir la dirección IP del nodo "master".

El comando finalizará mostrando instrucciones para configurar el acceso de un
usuario del sistema al cluster Kubernetes y también para añadir nodos al 
cluster.

El usuario se configurará copiando el fichero de la configuración del cluster
en el directorio del usuario:

    :::bash
    sudo cp - /etc/kubernetes/admin.conf ~/.kube/config

Recordar cambiar los permisos del fichero.

Para añadir un nodo al cluster, entramos en la máquina en cuestión y ejecutamos
el comando:

    :::bash
    kubeadm join --token 192.168.56.2:6443 --discovery-token-ca-cert-hash sha256:<hash>

Como se ha mencionado, estas instrucciones estarán disponibles al finalizar el
comando `kubeadm init`.
