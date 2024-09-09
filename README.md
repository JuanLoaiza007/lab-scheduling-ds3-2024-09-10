Esta es una practica del laboratorio de Schedulin de este [enlace](https://docs.google.com/document/d/1sD2kF8JtkHItLMnrcqdXgLyzadWIYvvQglA3lVlN0y4/edit).

## Metodo con Minikube

### Creación y activación de cluster

En este caso vamos a crear 3 nodos, un nodo será el control-plane y los otros serán workers (esto lo asigna automáticamente minikube).

1.  Crear un cluster de Kubernetes con 3 nodos

        minikube start --nodes=3 -p <profile_name>

    - **profile_name** es el nombre que se le dará al perfil (cluster) que se creará en minikube.

2.  Comprobar clusters con minikube

        minikube profile list

    - **Active Profile** es el contexto que esta usando minikube.
    - **Active Kubecontext** es el contexto que esta usando kubectl.

3.  Para usar el perfil indicado en minikube sin necesariamente iniciarlo.

        minikube profile <profile_name>

4.  En caso de que no estuviese iniciado se puede usar este comando:

        minikube start --profile <profile_name>

    > [!NOTE]
    >
    > Si el perfil no existe se creara uno nuevo.

5.  Para listar los cluster (contexts) disponibles en kubectl se usa:

        kubectl config get-contexts

6.  Para activar el cluster (context) indicado en kubectl se usa:

        kubectl config use-context <context_name>

7.  Compruebe tanto en minikube como en kubectl que el cluster que se ha activado es el mismo.

        minikube profile list && kubectl config get-contexts

### Configuración del cluster

1.  Al nodo control plane se debe configurar para que solo reciba pods de trato "NoSchedule".

        kubectl taint nodes <node_name> node-role.kubernetes.io/control-plane=:NoSchedule

    > [!NOTE]
    >
    > El taint "NoSchedule" hace que el nodo solo recibe nodos de ese mismo tipo.

2.  Asignar la etiqueta el entorno de pruebas en el worker 1 y de entorno de producción en el worker 2.

        kubectl label nodes <node_worker01_name> environment=testing

        kubectl label nodes <node_worker02_name> environment=production

3.  Describe todos los nodos para comprobar que se han aplicado los taint y los labels.

        kubectl describe nodes

### Prueba de worker en entorno de pruebas con nginx

1.  Crea estos pods de prueba:

        kubectl run test-pod1 –image nginx
        kubectl run test-pod2 –image nginx
        kubectl run test-pod3 –image nginx
        kubectl run test-pod4 –image nginx
        kubectl run test-pod5 –image nginx

2.  Revisa donde se estan ejecutando, observa que ninguno se esta ejecutando en el control plane.

        kubectl get pods -o wide

3.  Elimina los pods

        kubectl delete pod test-pod1 test-pod2 test-pod3 test-pod4 test-pod5

### Aplicar taint al entorno de producción

1.  Aplicar taint al entorno de producción en el worker 2.

        kubectl taint nodes <node_name> environment=production:NoSchedule

2.  Vuelve a crear los pods y revisa donde quedaron:

        kubectl run test-pod1 –image nginx
        kubectl run test-pod2 –image nginx
        kubectl run test-pod3 –image nginx
        kubectl run test-pod4 –image nginx
        kubectl run test-pod5 –image nginx

        kubectl get pods -o wide

    > [!TIP]
    >
    > Los pods se han creado y asignado solo en el worker 1 porque es el único que no tiene taints.

### Afinidad de Nodo

1.  Aplica el pod de producción

        kubectl apply -f production-pod.yaml

2.  Revisa donde quedaron los pods

        kubectl get pods -o wide
