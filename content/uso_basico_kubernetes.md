Title: Uso basico de Kubernetes
Date: 2020-03-09 10:03
Category: Contenedores
Tags: kubernetes,configuracion
Slug: uso-basico-kubernetes
Authors: José L. Patiño-Andrés
Summary: Ejemplos de uso basico de Kubernetes.

### Pods

Definicion de un Pod en fichero Yaml:

    :::yaml
    apiVersion: v1
    kind: Pod

    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end

    spec:
      containers:
        - name: nginx-container
          image: nginx

Comandos:

- `kubectl create -f pod-definition.yaml`
- `kubectl get pods`
- `kubectl describe pods`
- `kubectl delete pod myapp-pod`

### Replica Sets

Definicion de un Replica Set en fichero Yaml:

    :::yaml
    apiVersion: apps/v1
    kind: ReplicaSet

    metadata:
      name: myapp-replicaset
      labels:
        app: myapp
        type: front-end
    
    spec:
      # Here we define the pod we are going to start and replicate.
      template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
          containers:
            - name: nginx-container
              image: nginx

      replicas: 3

      selector:
        matchLabels:
          type: front-end
    
Comandos:

- `kubectl create -f replicaset-definition.yaml`
- `kubectl get replicaset`
- `kubectl describe replicaset`
- `kubectl replace -f replicaset-definition.yaml` para aplicar modificaciones
  en un fichero `replicaset-definition.yaml` a un Replica Set en marcha.
- `kubectl scale --replicas=6 replicaset myapp-replicaset` para escalar un
  Replica Set en marcha a 6 maquinas - despues de esto, la definicion del
  fichero `replicaset-definition.yaml` prevalecera.
- `kubectl delete replicaset myapp-replicaset`

### Deployments

Definicion de un Deployment en fichero Yaml:

    :::yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-deployment
      labels:
        app: myapp
        type: front-end
    
    spec:
      template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
        spec:
          containers:
            - name: nginx-container
              image: nginx:1.12
    
      replicas: 3
    
      selector:
        matchLabels:
          app: myapp

Comandos:

- `kubectl create -f deployment-definition.yaml`
- `kubectl get deployments`
- `kubectl apply -f deployment-definition.yaml` actualizar un Deployment
  existente con cambios hechos a un fichero `deployment-definition.yaml`.
- `kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1` para cambiar 
  la imagen del contenedor en un Deployment en ejecucion.
- `kubectl rollout status deployment/myapp-deployment` para comprobar el estado 
  de los "rollouts".
- `kubectl rollout history deployment/myapp-deployment` para ver el historico de
  "rollouts".
- `kubectl rollout undo deployment/myapp-deployment` para revertir un Deployment 
  a un estado anterior.

### Services

Definicion de un Service en fichero Yaml:

    :::yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: back-end
    
    spec:
      type: ClusterIP
      ports:
        - targetPort: 80
          port: 80
    
      selector:
        app: myapp
        type: back-end

Tipos:

- ClusterIP
- NodePort
- LoadBalancer

Comandos:

- `kubectl create -f service-definition.yaml`
- `kubectl get services`
- `kubectl describe service back-end`
- ...
