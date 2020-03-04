Title: Cluster de VMs con VirtualBox
Date: 2020-03-03 18:30
Category: Virtualizacion
Tags: cluster,vm,virtualbox,kubernetes
Slug: virtualbox-cluster
Authors: Jose L. Patino-Andres
Summary: Creacion de un cluster de maquinas virtuales Ubuntu con VirtualBox.

### Motivacion

En determinados entornos, por ejemplo y especialmente si nos vemos obligados a
trabajar en sistemas Windows, puede ser necesario desplegar un pequeno cluster
de maquinas virtuales Linux para probar conceptos, aprender nuevas tecnologias o
configurar "laboratorios" para experimentar o entrenar habilidades.

Aqui vamos a describir como se puede crear un pequeno cluster de 3 maquinas
virtuales basadas en Ubuntu Server 18.04 utilizando VirtualBox.

Las maquinas estaran disponibles tanto a traves de una direccion IP dinamica
asignada por el mismo router al que se conecta el host Windows como a traves de
una IP estetica que asignaremos nosotros y que las mantendra efectivamente en 
una subred desde la que podran comunicarse unas con otras.

Esto puede sernos util para desplegar un cluster Kubernetes donde una de estas
maquinas sea el nodo principal o "master" y las otras dos sean nodos disponibles
a traves de la subred en los cuales desplegar "pods" de aplicaciones.

### Sistema y versiones

- Windows 10 Pro (version 1903), maquina anfitrion.
- VirtualBox 6.1.4.
- Ubuntu Server 18.04.03 (64 bit), maquinas virtuales.

### Creacion de las maquinas virtuales

Para obtener la imagen preconfigurada de una maquina virtual Ubuntu Server 
18.04, podemos descargarnos el fichero `.vdi` de [osboxes.org](https://www.osboxes.org/ubuntu-server/).

Una vez nos hemos descargado el fichero, creamos una nueva maquina con la GUI 
de VirtualBox. **Es importante configurar una memoria RAM de al menos 2048MB y
usar el fichero `.vdi` que hemos descargado de osboxes.org como disco virtual**.

![New VM name and type](images/vbox_cluster_new_vm_name_type.png)

![New memory size of at least 2GB](images/vbox_cluster_new_vm_memory.png)

![New VM hard disk](images/vbox_cluster_new_vm_hard_disk.png)

Cuando ya hemos creado la primera maquina virtual, pasamos a la configuracion de
red donde marcamos "Bridged Adapter", de modo que la maquina virtual utilizara
la tarjeta de red asignada en la maquina anfitrion Windows 10 como gateway.

![Configure basic VM networking](images/vbox_cluster_network_bridged.png)

Una vez hecho esto, podemos clonar la maquina virtual en dos maquinas mas.

![Clone VM into Node 1](images/vbox_cluster_clone_vm.png)

Debemos hacer un clon "link", no un clon "copia". De este modo ahorramos
espacio de disco.

![Cloned VM config](images/vbox_cluster_clone_vm_settings.png)

Repetimos el mismo paso para la siguiente maquina virtual. Al final, tendremos
3 maquinas virtuales: la original ("master") y los dos clones ("nodos").

### Configuracion de subred

Para configurar una subred y asignar IPs estaticas a nuestras maquinas
virtuales de modo que siempre puedan comunicarse unas con otras, primero vamos
al menu "Tools" de VirtualBox, seleccionando la opcion "Network".

![VirtualBox Tools-Network](images/vbox_cluster_network_settings.png)

Tenemos que asegurarnos de que la opcion "DHCP Server" aparece **desmarcada**.

Ahora vamos otra vez a la configuracion de red de la maquina virtual, donde
anadimos un nuevo adaptador de red "Adapter 2", seleccionando para este la
opcion "Host-only Adapter" y relacionandolo con el adaptador del host donde
antes habiamos deseleccionado la opcion del servidor DHCP.

![New VM 2nd network adapter](images/vbox_cluster_network_host_only.png)

Repetimos este mismo paso para las dos maquinas clones.

Ahora estamos listos para arrancar las maquinas.

### Configuracion de sistema de maquinas virtuales

Recordar que debemos instalar `openssh-server` en las maquinas para asi poder
acceder a ellas desde el `cmd` de Windows 10. Esto nos permitira ejecutar las
maquinas en modo *headless*.

Ahora debemos configurar la red en las maquinas virtuales. Esto se hace mediante
la aplicacion de sistema `netplan`. Para ello, creamos un nuevo fichero
`/etc/netplan/01-netcfg.yaml`, con el siguiente contenido:

    :::yaml
    network:
      version: 2
      renderer: networkd
      ethernets:
        enp0s3:
          dhcp4: true
        enp0s8:
          dhcp4: false
          addresses: [192.168.56.2/24]
          gateway4: 192.168.56.1
          nameservers:
            addresses: [8.8.8.8,8.8.4.4]

Con esto lo que hacemos es configurar 2 tarjetas de red: la primera sera 
`enp0s3`, que recibe una IP dinamica mediante DHCP desde el router al que se
conecta el anfitrion Windows, y la segunda `enp0s8` que tiene una IP estatica
que como vemos esta en el rango de direcciones disponible configurado en el
adaptador de red de VirtualBox. Vamos a darle a esta primera maquina la IP
estatica `192.168.56.2` y a configurar sus DNS y su gateway.

Ahora podemos probar la configuracion ejecutando el siguiente comando:

    sudo netplan try

Si todo ha ido bien, ahora podemos guardar esta configuracion de red:

    sudo netplan apply

Tenemos que repetir el mismo proceso para las dos siguientes maquinas virtuales,
pero poniendo por supuesto diferentes IPs dinamicas. Por ejemplo `192.168.56.3`
y `192.168.56.4`.

En este momento ya deberiamos tener las maquinas completamente configuradas.

### Arranque "headless" de maquinas virtuales

En lugar de utilizar la GUI de VirtualBox, ejecutando `VBoxManage` desde la
`cmd` de Windows podemos arrancar las maquinas virtuales sin interfaz grafica:

    "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm <NOMBRE> --type headless
