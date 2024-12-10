# Workshop-OKE-Autoscaling

### Requerimientos:

- Cuenta de Oracle Cloud Infrastructure(test gratuito https://www.oracle.com/cloud/free/)
- Cuenta de Github (https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home)
- Poseer servidor linux con las siguientes dependencias instaladas:
  - git
  - python36-oci-cli
  - podman - docker 


### Paso a Paso

1. Crear Cuenta en git, usuar correo empresarial o personal (github.com)


https://user-images.githubusercontent.com/14284928/219779574-ad656998-a63d-402a-87cd-b9658c84e401.mov


2. Creación de compartment con el nombre de cada uno ej: **fbasso**
```
Menú Principal > Identity & Security > Compartments > Create Compartment
```
https://github.com/user-attachments/assets/b5a06579-a0a5-49b2-ab0c-cf1f8cd406c7


3. Creación de cluster OKE
```
Menú Principal > Developer Services > Kubernetes Cluster (OKE) > Create cluster > Quick create
Nombre de cluster: nombreUsusario
Compartemnt: El que se creó en pasosos anteriores
Kubernetes API endpoint: Public endpoint
Node type: Managed
Kubernetes worker nodes: Private Workers
Shape and image:
                  OCPU: 3
                  Memory: 8 
Node count: 1
```

https://github.com/user-attachments/assets/915785dd-ff2a-48cb-b924-927470d156f8

4. Creación de Registry con el nombre app1 y el nombre de usuario, ej: app1fbasso
```
Menú Principal > Developer Services > Container Registry > Create Repository
```
https://github.com/user-attachments/assets/5f22c213-958d-40af-9fd3-231a42eac5f4

```
Una vez creado el registry, copiar el namespace
```
![image](https://github.com/user-attachments/assets/35bfa005-7806-4afc-ac9a-d88bf2bc8e99)


5. Crear Token para conexión a registry
```
Esquina superior derecha
Profile > User Setting > Auth tokens > Generate Token
El token debe ser copiado y almacenado, ya que no se puede borrar
```
https://github.com/user-attachments/assets/2dc9a168-f2c9-45b7-83e9-2590944e6a41


6. Creación de imágen y subirla a registry, ejecutar los siguientes comandos:
```
CREAR IMÁGEN
$ cd Docker/
$ podman build --tag NOMBRE_REGISTRY -f Dockerfile                  # En mi caso es podman build --tag app1fbasso:latest -f Dockerfile
$ podman images
```

https://github.com/user-attachments/assets/ed6119b4-a43c-46f2-b865-d01e4f7681c4

```
SUBIR IMAGEN
$ podman login fra.ocir.io
        Usuario: NAMESPACE/USUARIO
        Pass: token creado previamente
$ podman tag localhost/app1fbasso:latest fra.ocir.io/frdpzymjc7jw/app1fbasso:latest
$ podman push fra.ocir.io/frdpzymjc7jw/app1fbasso:latest
```

https://github.com/user-attachments/assets/c4bc8823-1e63-4226-8f90-bb6dc94754ed

6. Una vez creado el cluster, se debe habilitar la opción de Autoscalin
```
Copiar OCID de node pool
Menú Principal > Developer Services > Kubernetes Cluster (OKE) > Cluster creado > Node pool > pool1
```
<img width="927" alt="image" src="https://github.com/user-attachments/assets/efbe02f7-71be-44f7-932c-d6621c057978">

```
Posterior a esto, ir a la opción de Add-ons > Manage add-ons
Buscar la opción Cluster Autoscaler y habilitar
Mantener la opción "nodes" y en value definir 1:5:ocid1.nodepool.oc1.XXXXXXX

Esto quiere decir que trabajaremos con un mínimo de 1 nodo y un máximo de 5 en el node pool recién creado (referenciado con su OCID) 
```

<img width="308" alt="image" src="https://github.com/user-attachments/assets/2e3c9e30-1ec5-41bd-bfc1-710fff235be4">

https://github.com/user-attachments/assets/070ee1b4-29f8-44ce-8dc4-01deb4e9964a

7. Cuando la configuración de Autoscaler esté habilitada y configurado acceder al cluster OKE
```
haciendo click en "Access Cluster" y la opción Cloud Shell Access, copiamos el comando de acceso y click en "Launch Cloud Shell"
Validamos el acceso a este con el comando
  kubectl get nodes
En mi caso ocupo el comando "k", que es un alias de kubectl
```
https://github.com/user-attachments/assets/cc2ff877-fb15-4394-a8b8-f401ecc22025

Validamos la configuración del autoscaling con el comando
```
kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS      AGE
cluster-autoscaler-7865498cb7-xsqgr    1/1     Running   0             28m
coredns-7d849c6df7-shg6h               1/1     Running   0             68m
csi-oci-node-tdg5z                     1/1     Running   1 (64m ago)   66m
kube-dns-autoscaler-57677746dd-wb8wq   1/1     Running   0             68m
kube-proxy-k46lb                       1/1     Running   0             66m
proxymux-client-zr5d8                  1/1     Running   0             66m
vcn-native-ip-cni-wjzbz                1/1     Running   0             66m
```
Como podemos validar, se creó el pod **cluster-autoscaler-7865498cb7-xsqgr** que, en este caso, es el encargado de mantener autoscaling 

8. Copiamos este yaml y lo desplegamos en cloud shell
```
---
apiVersion: v1
kind: Namespace
metadata:
  name: test
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apptest-dp
  labels:
    app: apptest
  namespace: test
spec:
  selector:
    matchLabels:
      app: apptest
  replicas: 3
  template:
    metadata:
      labels:
        app: apptest
    spec:
      containers:
      - name: apptest
        image: XXXXXXXXXXXXXXXXXXXXX
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
            memory: 800Mi
          requests:
            cpu: 400m
            memory: 500Mi
```
Este creará un namespace llamado _test_ y una aplicación llamada _apptest_
En el campo image, se puede definir la creada, en caso de no tener una imagen, usar la siguiente: fra.ocir.io/frdpzymjc7jw/apptest:latest

Una vez creado el archivo, aplicarlo en cluster
```
kubectl apply -f dp.yaml
```

https://github.com/user-attachments/assets/a7306689-5dbd-4ad2-901d-922619bb1107

Para validar la correcta recaión ejecutar

```
kubectl get all -n test
NAME                             READY   STATUS    RESTARTS   AGE
pod/apptest-dp-cc6b58886-26vr8   1/1     Running   0          6s
pod/apptest-dp-cc6b58886-2h2gr   1/1     Running   0          6s
pod/apptest-dp-cc6b58886-wpcsx   1/1     Running   0          6s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/apptest-dp   3/3     3            3           7s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/apptest-dp-cc6b58886   3         3         3       7s
```
<img width="497" alt="image" src="https://github.com/user-attachments/assets/d4f3852e-f9f8-44fa-abc7-2a65545ce493">

9. Para validar el autoscaler aumentar en réplicas el despliegue

Revisar la cantidad de nodos en el nodepool

<img width="886" alt="image" src="https://github.com/user-attachments/assets/a021cbff-2d03-4078-a576-66aa879a528f">

Ejecutar la escalación de pods
```
kubectl scale deployment apptest-dp --replicas=300 -n test
```
<img width="950" alt="image" src="https://github.com/user-attachments/assets/8f411c8c-a918-4555-a5ae-d7ab2b382236">


10.  
11. asd
12. add
13. das
14. asddas
15. das
16. das
17. asd
18. asd
19. asd
20. asd
21. das
22. sda
23. asd
24. das
25. das
