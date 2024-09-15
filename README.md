# Documentación Challenge
Durante la fase de desarrollo del Challenge 1, se trabajará con Minikube. Utilizaremos un clúster compuesto por 4 nodos, incluyendo el nodo de control plane.


## Arquitectura del clúster
El clúster está compuesto por los siguientes nodos:

* minikube: Nodo de control plane.
* minikube-m02: Nodo sin restricciones (sin taints).
* minikube-m03: Nodo aislado para evitar la programación de pods regulares.
* minikube-m04: Nodo sin restricciones (sin taints).



## Challenge 1
### Objetivo 1: Isolate speciﬁc node groups forbidding the pods scheduling in this node groups.

Para este challenge, aislaremos determinados nodos en el clúster utilizando taints, de manera que se controlará en qué nodos pueden ejecutarse los pods.

### 1. Creación del clúster 

Comando ejecutado para la creación del clúster con 4 nodos:

**minikube start --nodes 4 --driver=docker --memory 1900 --cpus 2**

### 2. Aplicación de taints

Comando taint nodo aleatorio (minikube-m03):

**kubectl taint nodes minikube-m03 special=isolated:NoSchedule**

Comando taint control-plane(minikube):

**kubectl taint nodes minikube node-role.kubernetes.io/control-plane=:NoSchedule**

Imagen que muestra que nodos tienen taints:

![Alt text](./images/taints.png)

### 3. Verificación de los taints aplicados

Comando para ver a que nodo pertenece cada pod:

**kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints**

![Alt text](./images/pods_node.png)

Modificando el replicaCount en el fichero values.yaml, se ve como se respeta el taint añadido en ambos nodos.
Con esta configuración terminamos el primer punto del challenge 1.

### Objetivo 2:  Ensure that a pod will not be scheduled on a node that already has a pod of the same type.

Para evitar tener pods repetidos en un mismo nodo vamos ha hacer uso de la Anit-Affinity.
Aumentamos el numero de nodos del clúster a 5:

#### minikube node add

cambios realizados en deployment.yaml para impedir que pods iguales se ejecuten en el mismo nodo:

    affinity:
    podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector: {}
            topologyKey: "kubernetes.io/hostname"

Para comprobar el correcto funcionamiento pondremos el replicaCount a 5:
![Alt text](./images/pods_affinity.png)

Al tener solamente 3 nodos libres solamente podemos tener 3 pods.

Tras reducir el replicaCount a 3, los pods pueden distribuirse adecuadamente entre nuestros nodos:
![Alt text](./images/pods_affinity_fine.png)