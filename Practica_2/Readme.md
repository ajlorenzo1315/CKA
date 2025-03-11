usar [google shell](https://shell.cloud.google.com/)

```bash 

# Esto iniciará un clúster de Kubernetes con 2 nodos (1 maestro y 1 worker) en la misma máquina.
minikube start --nodes=2 --driver=docker

# ip 10.88.0.3

minikube node add


```

# creamos un pod 

en el examaen controlador de ingreso usando engynext normalemte los clientes usan trafic o haproxsi eñ hproxi solo deja conectar  el node port con cualquier tipo de aplicaciones tiene muchas ventajas 
```bash

ajlorenzo_1315@cloudshell:~/CKA$ kubectl create -f po.yaml 
pod/multi-container-pod created
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get no                                                                                                                                                   
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   25m   v1.32.0
ajlorenzo_1315@cloudshell:~/CKA$ vi po.yaml                                                                                                                                                       
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get po
NAME                  READY   STATUS    RESTARTS   AGE
multi-container-pod   2/2     Running   0          4m59s
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get po multi-container-pod
NAME                  READY   STATUS    RESTARTS   AGE
multi-container-pod   2/2     Running   0          5m17s

```

# Pod 

```yml

apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
  labels:
    app: multi-container
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html
          
    - name: busybox
      image: busybox
      command: ["/bin/sh", "-c", "echo 'Hola desde BusyBox' > /data/index.html && sleep 3600"]
      volumeMounts:
        - name: shared-data
          mountPath: /data

  volumes:
    - name: shared-data
      emptyDir: {}
```

para conectarse. La red es virtual es como una red privadad solo de los pod que permite comunicarse entre ellos  

```bash

ajlorenzo_1315@cloudshell:~/CKA$ kubectl get po
NAME                  READY   STATUS    RESTARTS   AGE
multi-container-pod   2/2     Running   0          6m14s
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get po -owide
NAME                  READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
multi-container-pod   2/2     Running   0          6m23s   10.244.0.3   minikube   <none>           <none>
ajlorenzo_1315@cloudshell:~/CKA$ ip r
default via 10.88.0.1 dev eth0 
10.88.0.0/16 dev eth0 proto kernel scope link src 10.88.0.3 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
192.168.49.0/24 dev br-c113b5f4efe4 proto kernel scope link src 192.168.49.1 
ajlorenzo_1315@cloudshell:~/CKA$ kubectl exec multi-container-pod -- curl -sI 10.244.0.3
Defaulted container "nginx" out of: nginx, busybox
HTTP/1.1 200 OK
Server: nginx/1.27.4
Date: Tue, 11 Mar 2025 15:56:13 GMT
Content-Type: text/html
Content-Length: 19
Last-Modified: Tue, 11 Mar 2025 15:47:18 GMT
Connection: keep-alive
ETag: "67d05b06-13"
Accept-Ranges: bytes

# esto no esta bien en produción ya que puedes editar algo y que se reecree un pod
ajlorenzo_1315@cloudshell:~/CKA$ kubectl edit multi-container-pod
-bash: kurl: command not found

```
# recreate 

```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
  labels:
    app: nginx
spec:
  replicas: 3
  selector: # Que etiqueta va tener mis replicas 
    # se repite la etiqueta dos veces (aunque parece estupido) ese valor no tiene por que se igual
    # aqui se podria poner distinto al labels del template, el selector va tener todas las replicas 
    # como minimo selector 
    app: nginx
  template: # los pod vienen defenidos dentro del template en la api lle llama pod-template
  # no lleva nombre objeto dinamico se le añade de forma dinamica se le va añadir mas tarde 
  # tiene todo menos el nombre 
    metadata:
      labels:
        # estas son opcional por si quieres poner mas etiquetas
        app: nginx
        author: Sebas
        fecha: "2025" # str
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

**Nota** : si no se usan comillas en el label indica que es un int y puede dar el siguiente error 

```bash

Error from server (BadRequest): error when creating "rc.yaml": ReplicationController in version "v1" cannot be handled as a ReplicationController: json: cannot unmarshal number into Go struct field ObjectMeta.spec.template.metadata.labels of type string

```

**Nota: segun el profesor no se puede escalar los pods pero el profe lo va mirar**: el pod es imutable se puede modificar algunas cosas pero  limitado (como por ejemplo la etiqueta).

```bash 
kubectl scale --replicas 3 multi-container-pod
error: the server doesn't have a resource type "multi-container-pod"
```


# para hablar con las replicas usamos expose 

```bash 

kubectl get po -owide
NAME                  READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
multi-container-pod   2/2     Running   0          45m   10.244.0.3   minikube   <none>           <none>
nginx-rc-6q2sz        1/1     Running   0          18m   10.244.0.4   minikube   <none>           <none>
nginx-rc-7f2vk        1/1     Running   0          18m   10.244.0.6   minikube   <none>           <none>
nginx-rc-nq4xd        1/1     Running   0          18m   10.244.0.5   minikube   <none>           <none>


ajlorenzo_1315@cloudshell:~/CKA$ kube expose nginx-rc
-bash: kube: command not found
ajlorenzo_1315@cloudshell:~/CKA$ kubectl expose nginx-rc
error: the server does not have a resource type 'nginx-rc' 
ajlorenzo_1315@cloudshell:~/CKA$ kubectl expose rc nginx-rc
service/nginx-rc exposed

# crea una ip distinta a los de los pod 

# ip virtual en rangos ditintos 10.244.0.5(rango de red pod) 10.97.14.40(rango de servicio que hace un redireccionamiento a las ip de pods) no se puede ver la iptables , 
kubectl get svc nginx-rc
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
nginx-rc   ClusterIP   10.97.14.40   <none>        80/TCP    109s

(normalente si no son minikube si kubeproxy si se puede ver normalmente), pero si te metes dentro nodo puede ver la de iptable 

ajlorenzo_1315@cloudshell:~/CKA$ minikube ssh
docker@minikube:~$ iptables -S|grep 
Fatal: can't open lock file /run/xtables.lock: Permission denied
Usage: grep [OPTION]... PATTERNS [FILE]...
Try 'grep --help' for more information.
docker@minikube:~$ iptables -S|grep  10
Fatal: can't open lock file /run/xtables.lock: Permission denied
docker@minikube:~$ sudo iptables -S|grep  10
-A CNI-FORWARD -d 10.244.0.2/32 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A CNI-FORWARD -s 10.244.0.2/32 -j ACCEPT
-A CNI-FORWARD -d 10.244.0.3/32 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A CNI-FORWARD -s 10.244.0.3/32 -j ACCEPT
-A CNI-FORWARD -d 10.244.0.6/32 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A CNI-FORWARD -d 10.244.0.4/32 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A CNI-FORWARD -s 10.244.0.6/32 -j ACCEPT
-A CNI-FORWARD -s 10.244.0.4/32 -j ACCEPT
-A CNI-FORWARD -d 10.244.0.5/32 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A CNI-FORWARD -s 10.244.0.5/32 -j ACCEPT

# iptables 

docker@minikube:~$ sudo iptables -S -t nat|grep SVC.*default/nginx-rc
-A KUBE-SVC-Z3R7LL5CXYDH3WP6 ! -s 10.244.0.0/16 -d 10.97.14.40/32 -p tcp -m comment --comment "default/nginx-rc cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
-A KUBE-SVC-Z3R7LL5CXYDH3WP6 -m comment --comment "default/nginx-rc -> 10.244.0.4:80" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-XZ5RVJPRBT3TYSPE
-A KUBE-SVC-Z3R7LL5CXYDH3WP6 -m comment --comment "default/nginx-rc -> 10.244.0.5:80" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-2JSFUV72PQKD2HK6
-A KUBE-SVC-Z3R7LL5CXYDH3WP6 -m comment --comment "default/nginx-rc -> 10.244.0.6:80" -j KUBE-SEP-OYEEXILGJE3EWWH6
docker@minikube:~$ exit
logout


ajlorenzo_1315@cloudshell:~$ kubectl get ep nginx-rc
NAME       ENDPOINTS                                   AGE
nginx-rc   10.244.0.4:80,10.244.0.5:80,10.244.0.6:80   36m

```




**servicio es una regla de iptables  que vnix se encarga de mantener al dia**

hoy ya no ahora se usan flujos de virtual switch ls despliegues modernos iptables son del kernel y virtualwitch a espacio de usuario 


iptables y el kernel:

iptables es una herramienta tradicional para manejar reglas de filtrado de paquetes en Linux.
Funciona en el espacio del kernel, lo que significa que las reglas de firewall se aplican directamente en el núcleo del sistema operativo.
Aunque sigue usándose, en entornos modernos se ha reemplazado en gran medida por nftables, que es más eficiente y flexible.
Virtual Switch (vSwitch) y espacio de usuario:

En entornos modernos como Kubernetes, OpenStack y máquinas virtuales, los flujos de red no pasan directamente por iptables.
Se usan virtual switches como Open vSwitch (OVS), que operan en el espacio de usuario en lugar del kernel.
Esto permite una mayor flexibilidad y rendimiento en redes definidas por software (SDN).
OVS puede reenviar paquetes, aplicar reglas de filtrado y hacer NAT sin depender de iptables directamente.
Despliegues modernos:

En sistemas como Kubernetes, las reglas de red ya no se manejan con iptables, sino con eBPF (Extended Berkeley Packet Filter) y herramientas como Cilium.
Open vSwitch también se usa en entornos de virtualización como VMware NSX, OpenStack, Proxmox y Xen.
En estos entornos, los switches virtuales controlan cómo se enrutan los paquetes entre las máquinas virtuales o contenedores.


# 

```bash
# 
ajlorenzo_1315@cloudshell:~/CKA$ kubectl expose rc nginx-rc
service/nginx-rc exposed
ajlorenzo_1315@cloudshell:~/CKA$ 
service/nginx-rc exposed
-bash: service/nginx-rc: No such file or directory
ajlorenzo_1315@cloudshell:~/CKA$ service/nginx-rc exposed
-bash: service/nginx-rc: No such file or directory
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get svc nginx-rc
NAME       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx-rc   ClusterIP   10.99.238.32   <none>        80/TCP    19s
ajlorenzo_1315@cloudshell:~/CKA$ kubectl exec -c busybox multi-container-pod -- nslookup nginx-rc.default.svc.cluster.local | grep Name -A1
Name:   nginx-rc.default.svc.cluster.local
Address: 10.99.238.32


kubectl explain svc.spec.type
KIND:       Service
VERSION:    v1

FIELD: type <string>
ENUM:
    ClusterIP
    ExternalName
    LoadBalancer
    NodePort

DESCRIPTION:
    type determines how the Service is exposed. Defaults to ClusterIP. Valid
    options are ExternalName, ClusterIP, NodePort, and LoadBalancer. "ClusterIP"
    allocates a cluster-internal IP address for load-balancing to endpoints.
    Endpoints are determined by the selector or if that is not specified, by
    manual construction of an Endpoints object or EndpointSlice objects. If
    clusterIP is "None", no virtual IP is allocated and the endpoints are
    published as a set of endpoints rather than a virtual IP. "NodePort" builds
    on ClusterIP and allocates a port on every node which routes to the same
    endpoints as the clusterIP. "LoadBalancer" builds on NodePort and creates an
    external load-balancer (if supported in the current cloud) which routes to
    the same endpoints as the clusterIP. "ExternalName" aliases this service to
    the specified externalName. Several other fields do not apply to
    ExternalName services. More info:
    https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
    
    Possible enum values:
     - `"ClusterIP"` means a service will only be accessible inside the cluster,
    via the cluster IP.
     - `"ExternalName"` means a service consists of only a reference to an
    external name that kubedns or equivalent will return as a CNAME record, with
    no exposing or proxying of any pods involved.
     - `"LoadBalancer"` means a service will be exposed via an external load
    balancer (if the cloud provider supports it), in addition to 'NodePort'
    type. # Esto es solo si lo tiene el servicio estili amazon si no no hace nada suele cobrar por cada balanceo
    #  (en la nube, tipo AWS o GCP):
     - `"NodePort"` means a service will be exposed on one port of every node,
    in addition to 'ClusterIP' type.
    
# -----



ajlorenzo_1315@cloudshell:~/CKA$ kubectl get rc nginx-rc
NAME       DESIRED   CURRENT   READY   AGE
nginx-rc   3         3         3       3m30s
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get svc nginx-rc
NAME       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
nginx-rc   ClusterIP   10.99.238.32   <none>        80/TCP    3m20s
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get ep nginx-rc
NAME       ENDPOINTS                                   AGE
nginx-rc   10.244.0.4:80,10.244.0.5:80,10.244.0.6:80   3m24s

ajlorenzo_1315@cloudshell:~/CKA$ kubectl edit svc nginx-rc
error: services "nginx-rc" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-1653155382.yaml"
error: Edit cancelled, no valid changes were saved.
ajlorenzo_1315@cloudshell:~/CKA$ curl -sI localhost:31444
ajlorenzo_1315@cloudshell:~/CKA$ kubectl edit svc nginx-rc                                                                                                                          # editamos para que el tipo sea node prot no deja cambiarlo por consola de comandos              
service/nginx-rc edited
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get ep nginx-rc                                                                                                                                          
NAME       ENDPOINTS                                   AGE
nginx-rc   10.244.0.4:80,10.244.0.5:80,10.244.0.6:80   6m19s



# --- Este funciona si no existiene 

ajlorenzo_1315@cloudshell:~/CKA$ kubectl expose rc nginx-rc --type NodePort
Error from server (AlreadyExists): services "nginx-rc" already exists

### comprobar que se ha cambiado el estilo 


ajlorenzo_1315@cloudshell:~/CKA$ minikube ssh -- curl -sI localhost:31444
ssh: Process exited with status 7
ajlorenzo_1315@cloudshell:~/CKA$ kubectl exec nginx-rc -- curl -sI localhost
Error from server (NotFound): pods "nginx-rc" not found
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get svc nginx-rc
NAME       TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
nginx-rc   NodePort   10.99.238.32   <none>        80:31342/TCP   12m
ajlorenzo_1315@cloudshell:~/CKA$ kubectl exec nginx-rc -- curl -sI localhost
Error from server (NotFound): pods "nginx-rc" not found
ajlorenzo_1315@cloudshell:~/CKA$ kubectl get pod
NAME                  READY   STATUS    RESTARTS   AGE
multi-container-pod   2/2     Running   0          14m
nginx-rc-gdx2p        1/1     Running   0          13m
nginx-rc-qzxzt        1/1     Running   0          13m
nginx-rc-r4fm9        1/1     Running   0          13m
ajlorenzo_1315@cloudshell:~/CKA$ kubectl exec nginx-rc-gdx2p -- curl -sI localhost
HTTP/1.1 200 OK
Server: nginx/1.27.4
Date: Tue, 11 Mar 2025 18:39:34 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 05 Feb 2025 11:06:32 GMT
Connection: keep-alive
ETag: "67a34638-267"
Accept-Ranges: bytes

ajlorenzo_1315@cloudshell:~/CKA$ minikube ssh -- curl -sI localhost:31444
ssh: Process exited with status 7
ajlorenzo_1315@cloudshell:~/CKA$ minikube ssh -- curl -sI localhost:31342
HTTP/1.1 200 OK
Server: nginx/1.27.4
Date: Tue, 11 Mar 2025 18:40:02 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Wed, 05 Feb 2025 11:06:32 GMT
Connection: keep-alive
ETag: "67a34638-267"
Accept-Ranges: bytes

```
El comando:  
```bash
kubectl exec -c busybox multi-container-pod -- nslookup nginx-rc.default.svc.cluster.local | grep Name -A1
```
**📌 Explicación paso a paso:**

### **1️⃣ `kubectl exec`**
Ejecuta un comando dentro de un pod en Kubernetes.  

- `-c busybox` → Especifica que queremos ejecutar el comando dentro del **contenedor** llamado `busybox` dentro del pod.  
- `multi-container-pod` → Es el **nombre del pod** en el que se ejecutará el comando.  
- `--` → Indica que todo lo que sigue es el comando a ejecutar dentro del contenedor.  

### **2️⃣ `nslookup nginx-rc.default.svc.cluster.local`**
Busca la dirección IP del servicio `nginx-rc.default.svc.cluster.local` usando **nslookup** (una herramienta para resolver nombres DNS).  

- `nginx-rc.default.svc.cluster.local` → Es un **Servicio de Kubernetes**, donde:  
  - `nginx-rc` → Nombre del servicio.  
  - `default` → Espacio de nombres (namespace).  
  - `svc.cluster.local` → Dominio interno de Kubernetes para los servicios.  

💡 **Objetivo:** Verificar si el servicio `nginx-rc` está resolviendo correctamente a una IP dentro del clúster.  

### **3️⃣ `| grep Name -A1`**
Filtra la salida del comando `nslookup` para mostrar solo las líneas relevantes:  

- `grep Name` → Busca la línea donde aparece `Name:` (el nombre del dominio resuelto).  
- `-A1` → Muestra **una línea adicional después** del resultado encontrado (que normalmente será la IP del servicio).  

---

### **📌 Ejemplo de salida del comando**
Si `nginx-rc.default.svc.cluster.local` se está resolviendo correctamente, el comando podría devolver algo como:  

```
Name:   nginx-rc.default.svc.cluster.local
Address: 10.100.200.5
```
Esto confirma que el servicio **nginx-rc** tiene la IP **10.100.200.5** en el clúster de Kubernetes.  

---

### **✅ ¿Para qué se usa este comando?**
1. **Verificar la resolución de DNS** dentro del clúster.  
2. **Comprobar que un servicio de Kubernetes (`nginx-rc`) está disponible** y accesible por su nombre interno.  
3. **Depurar problemas de conectividad** cuando los pods no pueden comunicarse con un servicio.  

Si `nslookup` no devuelve una IP, significa que el servicio no está disponible o hay un problema de DNS en el clúster.  

---

🔹 **En resumen:**  
✔️ El comando **ejecuta `nslookup` dentro de un contenedor `busybox`** para verificar si el servicio `nginx-rc` tiene una IP asignada.  
✔️ **Filtra la salida** para mostrar solo el nombre y la IP del servicio.  
✔️ Es útil para **depurar problemas de conectividad** dentro de Kubernetes. 🚀
 
## Diferencia Entre Tipos de Servicios en Kubernetes

| Tipo           | Explicación                                           | Ejemplo de Uso                              |
|--------------|-----------------------------------------------------|--------------------------------------------|
| `ClusterIP`   | Accesible solo dentro del clúster                   | Comunicación entre microservicios internos |
| `NodePort`    | Expuesto en un puerto específico de cada nodo        | Acceder desde fuera sin un balanceador    |
| `LoadBalancer`| Usa un balanceador de la nube para exponer el servicio | Apps web en AWS/GCP/Azure                 |
| `ExternalName`| Redirige tráfico a otro dominio (CNAME)              | Conectar con un servicio externo          |



## localhost

solo un Node port (puede ri de 30000 a 32000) el node port apunta a un proxi lo hace atraves del node port , no es que el node por tenga un iptables  que apunta al proxi atraves del node port se conecta al proxi y entoces el proxi se conecta al backend y se lo devuelve al cliente con este truco puede tener miles de aplicaciones con un node port y esto es un controlador de ingraso 



### **¿Qué hace el `NodePort` en Kubernetes?**
Un servicio de tipo `NodePort` en Kubernetes **expone un puerto específico en todos los nodos del clúster**. Esto permite que los usuarios puedan acceder a la aplicación desde **cualquier nodo del clúster** a través de ese puerto.

📌 **Flujo de conexión con `NodePort`**:
1. Un usuario hace una petición a `<IP_DEL_NODO>:<NODE_PORT>`.
2. Kubernetes redirige la solicitud a un **proxy interno** (kube-proxy).
3. `kube-proxy` reenvía la solicitud a un **pod disponible en el backend**.
4. El pod procesa la solicitud y envía la respuesta de vuelta al cliente.

> ⚠️ **Importante**: `NodePort` expone la aplicación en **todos** los nodos del clúster, incluso si el pod no está corriendo en ese nodo.

---

### **Rango y Máximo de `NodePort`**
El puerto asignado para un servicio `NodePort` se encuentra dentro del **rango por defecto de 30000 a 32767**.

- **Máximo de `NodePort`**: 32767 - 30000 + 1 = **2768 posibles servicios `NodePort`**.
- Puedes modificar este rango en la configuración de Kubernetes (`--service-node-port-range=XXXX-YYYY` en `kube-apiserver`).

📌 **Ejemplo de uso de `NodePort`**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-app-service
spec:
  type: NodePort
  selector:
    app: mi-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30080  # Debe estar en el rango 30000-32767
```
🔹 Ahora puedes acceder a la app desde cualquier nodo en `http://<IP_DEL_NODO>:30080`.

---

### **Relación entre `NodePort` y el Controlador de Ingreso**
Cuando mencionaste:

> "NodePort apunta a un proxy a través del node port, el proxy se conecta al backend y devuelve la respuesta al cliente. Con este truco, se pueden tener miles de aplicaciones con un solo NodePort."

Se refiere a que **no es necesario exponer cada servicio directamente con un NodePort**. En su lugar, se puede usar un **controlador de ingreso (Ingress Controller)** que:
1. **Escucha en un solo NodePort** (Ejemplo: `30080`).
2. **Redirige las solicitudes** internamente a diferentes aplicaciones según el dominio o la URL.
3. **Hace de proxy inverso**, enviando las peticiones a los servicios adecuados dentro del clúster.

### **Ejemplo con un Controlador de Ingreso**
En lugar de exponer cada servicio con un `NodePort`, podemos usar un **Ingress** con Nginx:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mi-ingress
spec:
  rules:
    - host: mi-app.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: mi-app-service
                port:
                  number: 80
```
🔹 Ahora, cualquier solicitud a `http://mi-app.local` será redirigida al **servicio interno** sin necesidad de exponer múltiples `NodePorts`.

---

### **Resumen**
- `NodePort` **expone una aplicación en un puerto de todos los nodos**.
- Su rango por defecto es **30000-32767**, permitiendo hasta **2768 NodePorts**.
- **No usa directamente iptables**, sino que `kube-proxy` redirige el tráfico al pod correcto.
- En lugar de usar múltiples `NodePorts`, un **controlador de ingreso** (como Nginx o Traefik) puede recibir las peticiones y distribuirlas internamente. 🚀



### **Controlador de Ingreso en Kubernetes: Nginx, Traefik y HAProxy**  

En un examen sobre Kubernetes e Ingress Controllers, es importante conocer los diferentes controladores de ingreso como **Nginx, Traefik y HAProxy**, así como su interacción con `NodePort`.  

---

## **📌 ¿Qué es un Controlador de Ingreso en Kubernetes?**
Un **Ingress Controller** es un proxy que gestiona el tráfico externo hacia los servicios internos de un clúster de Kubernetes. En lugar de exponer múltiples `NodePort` o `LoadBalancer`, un **Ingress Controller** escucha en un **único NodePort** y luego redirige el tráfico según el dominio o ruta solicitada.  

---

## **🔥 Relación entre `NodePort` y los Controladores de Ingreso**
- Un **Ingress Controller** se expone usando un **Service de tipo `NodePort`** o `LoadBalancer`.
- Este `NodePort` **no está directamente expuesto a las aplicaciones**, sino que actúa como punto de entrada al clúster.
- Desde ahí, el controlador **redirige el tráfico** a los servicios internos.

📌 **Ejemplo con un Service de tipo `NodePort` para Ingress Controller**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
spec:
  type: NodePort
  selector:
    app: ingress-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080  # Nodo donde escuchará el Ingress Controller
```

### **Flujo de tráfico con NodePort y un Ingress Controller**
1. Un usuario accede a `http://mi-app.com`.
2. La petición llega al `NodePort` (Ejemplo: `30080`).
3. El **Ingress Controller (Nginx, Traefik, HAProxy, etc.)** recibe la solicitud.
4. Según la configuración de `Ingress`, el controlador **redirige el tráfico** al servicio correspondiente.
5. El pod de la aplicación responde y la respuesta vuelve al cliente.

---

## **🚀 Comparación entre Nginx, Traefik y HAProxy como Ingress Controller**
| Ingress Controller | Características Clave | Ventajas |
|--------------------|----------------------|----------|
| **Nginx** (más usado en exámenes) | Popular, fácil de configurar, ampliamente documentado | Soporte robusto y sencillo para la mayoría de casos |
| **Traefik** | Más dinámico, auto-descubrimiento de servicios, soporte nativo para gRPC | Ideal para arquitecturas **nativas en la nube** |
| **HAProxy** | Proxy de alto rendimiento, balanceo avanzado, optimización de tráfico | **Más eficiente** en términos de latencia y escalabilidad |

---

## **📌 ¿Por qué HAProxy solo se conecta con NodePort?**
Mencionaste que **HAProxy solo deja conectar con `NodePort`**. Esto es porque:
- HAProxy **necesita una entrada fija** al clúster.
- `NodePort` **expone un puerto en cada nodo**, permitiendo que HAProxy lo use como **punto de acceso**.
- Desde ahí, **balancea** y **redirecciona** el tráfico hacia los diferentes servicios internos.

📌 **Ejemplo de configuración de HAProxy para Kubernetes**:
```yaml
frontend kubernetes
    bind *:30080
    default_backend ingress

backend ingress
    server ingress-nginx 10.0.0.1:30080  # Direccionando al NodePort de Kubernetes
```

### **🔹 Ventajas de usar HAProxy con Kubernetes**
✅ Manejo eficiente de múltiples aplicaciones en un solo NodePort.  
✅ Balanceo avanzado y control total sobre la distribución de tráfico.  
✅ Reducción de costos, ya que no se necesitan múltiples `LoadBalancer`.  

---

## **✅ Resumen Final**
- Un **Ingress Controller** (como Nginx, Traefik o HAProxy) **usa un NodePort** para recibir tráfico externo.
- HAProxy **requiere un NodePort** porque no interactúa directamente con los servicios de Kubernetes.
- **Traefik** y **Nginx** pueden integrarse más dinámicamente con Kubernetes, pero HAProxy ofrece **mayor control y rendimiento**.
- Usar un **único NodePort** con un Ingress Controller permite **manejar múltiples aplicaciones** sin exponer cada servicio individualmente.

---