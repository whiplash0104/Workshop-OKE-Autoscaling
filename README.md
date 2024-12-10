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

7. Cuando la configuración de Autoscaler esté habilitada y configurada 

8.  
9. asd
10. add
11. das
12. asddas
13. das
14. das
15. asd
16. asd
17. asd
18. asd
19. das
20. sda
21. asd
22. das
23. das
