
En el espacio de nombres `multi-containers`, crea un pod llamado `containers-pod` que contenga 3 contenedores con las siguientes características:

1. **Contenedor `primero`**:  
   - Imagen: `nginx:1.17`  
   - Expone el puerto: `8080`

2. **Contenedor `segundo`**:  
   - Imagen: `busybox:1.28`  
   - Variable de entorno: `ORDER=SECOND`  

3. **Contenedor `tercero`**:  
   - Imagen: `busybox:1.31.1`  
   - Variable de entorno: `ORDER=THIRD` 

whith  [labs.play-with-k8s.com](https://labs.play-with-k8s.com)


```bash
# en un nodeo ejecuatmaos el admin
# 1. Initializes cluster master node 1:
kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
# 2. Initializes work in node 2:
#Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.11:6443 --token tvi0d4.da9x56110jmh11h0 \
        --discovery-token-ca-cert-hash sha256:2cd8cb146c8d348e6324ff75e2b3c00ca3cd36cf63ded4a778ba6f5c63a10071

# 3. Conetamos a traves de una red

$ kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml

configmap/kube-router-cfg created
daemonset.apps/kube-router created
serviceaccount/kube-router created
clusterrole.rbac.authorization.k8s.io/kube-router created
clusterrolebinding.rbac.authorization.k8s.io/kube-router created

```


```bash
$ kubectl get namespaces
kubectl get namespaces
NAME              STATUS   AGE
default           Active   25m
kube-node-lease   Active   25m
kube-public       Active   25m
kube-system       Active   25m

$ kubectl create namespace multi-containers

namespace/multi-containers created
```
```bash
# creamos la estructura de un pod 
kubectl run containers-pods --image=busybox:1.28 --env="ORDER=FIRST" --dry-run=client -o yaml > pod.yaml 

vi pod.yaml 

kubectl create -f pod.yaml 
``` 

```bash
# comprobamos que todo este bien 

kubectl get po -A | grep multi-containers
multi-containers   container-pod                   3/3     Running   0          2m44s


kubectl delete pod containers-pod -n multi-containers
kubectl delete namespace multi-container
```

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: container-pod
  namespace: multi-containers
spec:
  containers:
  - name: busybox-second
    image: busybox:1.28
    env:
    - name: ORDER
      value: SECOND
    command: ["sleep", "8000"]  # Se usa sleep en lugar de time.sleep()

  - name: nginx-container
    image: nginx:1.17
    ports:
    - containerPort: 8080

  - name: busybox-third
    image: busybox:1.31.1
    env:
    - name: ORDER
      value: THIRD

    command: ["sleep", "8000"] # Si no usamos un comando de espera se caen debido a que ejecuta la shell y termina similar a docker CrashLoopBackOff

  dnsPolicy: ClusterFirst
  restartPolicy: Always

```


# Profesor

```bash
apiVersion: v1
kind: Pod
metadata:
  name: tres-containers-pod
  namespace: ckad-multi-containers
spec:
  containers:
  - env:
    - name: ORDER
      value: FIRST
    image: busybox:1.28
    name: primero
    command:
    - sh
    - -c
    - sleep 30000000
  - image: nginx:1.17
    name: segundo
    ports:
    - containerPort: 8080
  - env:
    - name: ORDER
      value: THIRD
    image: busybox:1.31.1
    name: tercero
    command:
    - sh
    - -c
    - sleep 30000000
```


Chantgpt

¡Entendido! Aquí tienes una serie de ejercicios para practicar para el examen de certificación de Kubernetes.  

---

### 🔹 **Ejercicios sobre Pods**
1. **Crea un Pod con Volumen Compartido**  
   En el espacio de nombres `shared-volumes`, crea un Pod llamado `volume-pod` con dos contenedores:  
   - `writer`: Usa la imagen `busybox`, escribe la fecha y hora en un archivo dentro de un volumen compartido cada 5 segundos.  
   - `reader`: Usa la imagen `nginx`, expone el archivo como una página web en el puerto 80.  


```yaml

apiVersion: v1
kind: Pod
metadata:
    name: "volume-pod"
    namespace: "shared-volumes"
    labels:
        name: volume-pod  # <-- Etiqueta agregada para que el Service pueda encontrar este Pod
spec:
    volumes:
        - name: "shared-data"
          emptyDir: {}  # Volumen compartido entre los contenedores
    containers:
        -   name: busybox
            image: busybox:1.28
            env:
                - name: ORDER
                  value: SECOND
            command: ["/bin/sh", "-c"]
            args:
                - "while true; do
                    date >> /usr/share/nginx/html/index.html;
                    sleep 5;
                done;"
            volumeMounts:
                - name: shared-data
                  #mountPath: /data
                  mountPath: /usr/share/nginx/html/  # <-- Ahora escribe en la misma ruta que nginx

        -   name: nginx-container
            image: nginx:1.17
            ports:
                - containerPort: 80
            volumeMounts:
                - name: "shared-data"
                  mountPath: /usr/share/nginx/html/  # Sirve el archivo como página web
```
```bash
esto da un error de que shared-data pero si haco car a /data/si esxiste index en nginx-container
[node3 ~]$ kubectl exec -it volume-pod -n shared-volumes -c busybox -- ls -l /data
total 8
-rw-r--r--    1 root     root          4553 Mar 11 21:20 index.html
[node3 ~]$ kubectl exec -it volume-pod -n shared-volumes -- cat /usr/share/nginx/html/index.html
Defaulted container "busybox" out of: busybox, nginx-container
cat: cant open '/usr/share/nginx/html/index.html': No such file or directory
command terminated with exit code 1

#solucion  busybox

# mountPath: /usr/share/nginx/html  # <-- Ahora escribe en la misma ruta que nginx

kubectl delete pod volume-pod -n shared-volumes
kubectl apply -f volume-pod.yaml

```

```bash
kubectl get pods -n shared-volumes

# para acceder al contenido 

kubectl port-forward pod/volume-pod 8080:80 -n shared-volumes

# Luego abre en el navegador: http://localhost:8080

```


```yaml

apiVersion: v1
kind: Service
metadata:
  name: volume-service
  namespace: shared-volumes
spec:
  selector:
    name: volume-pod
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080

```

```bash
kubectl apply -f volume-service.yaml

[node3 ~]$ kubectl get svc -n shared-volumes
NAME             TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
volume-service   NodePort   10.106.58.186   <none>        80:30080/TCP   116s

pod "curl-test" deleted
# Verificar las etiquetas del Pod en el espacio de nombres shared-volumes
[node3 ~]$ kubectl get pods -n shared-volumes --show-labels
NAME         READY   STATUS    RESTARTS   AGE     LABELS
volume-pod   2/2     Running   0          4m31s   <none>
# El Pod no tenía la etiqueta necesaria para que el Service NodePort lo detectara, la añadimos manualmente
[node3 ~]$ kubectl label pod volume-pod -n shared-volumes name=volume-pod --overwrite
pod/volume-pod labeled
# solucion añadir el label names de manera manual para que nodeport tenga acceso 
# Verificar nuevamente que el Pod ahora tenga la etiqueta correcta
[node3 ~]$ kubectl get pods -n shared-volumes --show-labels
NAME         READY   STATUS    RESTARTS   AGE     LABELS
volume-pod   2/2     Running   0          4m45s   name=volume-pod
# Revisar si el Service ya está correctamente enlazado al Pod y tiene los Endpoints configurados
[node3 ~]$ kubectl get endpoints volume-service -n shared-volumes
NAME             ENDPOINTS     AGE
volume-service   10.5.1.4:80   18m

```
La **etiqueta correcta en el Pod** es necesaria porque el **Service en Kubernetes usa un selector de etiquetas** para encontrar los Pods a los que debe dirigir el tráfico. Si el Pod no tiene la etiqueta esperada, el Service **no lo detecta** y, por lo tanto, **no redirige el tráfico a él**.

---

### 🔍 **📌 Explicación con tu caso específico**
1. **El Service que creaste (`volume-service`) tiene este selector**:
   ```yaml
   selector:
     name: volume-pod
   ```
   Esto significa que el Service **solo enviará tráfico a los Pods que tengan la etiqueta `name=volume-pod`**.

2. **Tu Pod originalmente no tenía esta etiqueta**:
   ```sh
   NAME         READY   STATUS    RESTARTS   AGE     LABELS
   volume-pod   2/2     Running   0          4m31s   <none>  # <-- Falta la etiqueta name=volume-pod
   ```
   Como el Pod no tenía la etiqueta `name=volume-pod`, el Service **no lo encontraba** y no podía enviar tráfico a él.

3. **Después de agregar la etiqueta manualmente**:
   ```sh
   kubectl label pod volume-pod -n shared-volumes name=volume-pod --overwrite
   ```
   El Pod ahora tenía la etiqueta correcta:
   ```sh
   NAME         READY   STATUS    RESTARTS   AGE     LABELS
   volume-pod   2/2     Running   0          4m45s   name=volume-pod  # <-- Ahora sí coincide con el selector
   ```

4. **Esto hizo que el Service encontrara el Pod y estableciera un `ENDPOINT`**:
   ```sh
   kubectl get endpoints volume-service -n shared-volumes
   ```
   Salida esperada:
   ```
   NAME             ENDPOINTS     AGE
   volume-service   10.5.1.4:80   18m
   ```
   Ahora el Service `NodePort` sabe que debe redirigir las peticiones a `10.5.1.4:80` (que es la dirección interna del Pod `volume-pod`).

---
🔥 **📌 Resumen**
✅ **Sin la etiqueta correcta** → El Service **no encuentra** el Pod y **no redirige tráfico**.  
✅ **Con la etiqueta correcta** → El Service ahora **sabe a dónde enviar las peticiones**.  
✅ **Los `ENDPOINTS` deben aparecer** en `kubectl get endpoints` para confirmar que el Service y el Pod están conectados.


2. **Pod con un Init Container**  
   Crea un Pod en el namespace `init-example` llamado `init-pod` con un Init Container que:  
   - Usa `busybox`, imprime "Preparando aplicación" y espera 5 segundos antes de terminar.  
   - Luego, un contenedor `nginx` se ejecuta normalmente.  

---

### 🔹 **Ejercicios sobre Deployments**
3. **Escalabilidad y Actualización de un Deployment**  
   - Crea un Deployment en el namespace `scaling-app` llamado `my-deployment` con 3 réplicas usando `nginx:1.18`.  
   - Luego, escala el deployment a 5 réplicas.  
   - Posteriormente, actualiza la imagen a `nginx:1.19`.  
   - Revisa que no haya downtime durante la actualización.  

4. **Configura un Rolling Update con Estrategia Blue-Green**  
   - Crea un Deployment `blue-green` en el namespace `rollout-example` con la imagen `nginx:1.18`.  
   - Usa `kubectl rollout` para cambiar la imagen a `nginx:1.20` y revisa el progreso.  
   - Si hay problemas, revierte el cambio a la versión anterior.  

---

### 🔹 **Ejercicios sobre Servicios y Networking**
5. **Configurar un Service para Exponer un Pod**  
   - Crea un Pod en el namespace `network-test` llamado `web-pod` con `nginx` en el puerto 80.  
   - Luego, crea un Service de tipo ClusterIP llamado `web-service` que lo exponga en el puerto 8080.  
   - Usa `kubectl port-forward` para acceder al servicio desde tu máquina.  

6. **Exponer un Deployment con un LoadBalancer**  
   - Crea un Deployment en el namespace `loadbalancer-test` llamado `web-deploy` con 2 réplicas usando la imagen `httpd:2.4`.  
   - Exponlo mediante un Service de tipo LoadBalancer en el puerto 80.  
   - Verifica que puedes acceder a la aplicación desde fuera del clúster.  

---

### 🔹 **Ejercicios sobre ConfigMaps y Secrets**
7. **Usar ConfigMaps para Configurar un Pod**  
   - Crea un ConfigMap llamado `app-config` en el namespace `config-example` con una clave `APP_MODE=production`.  
   - Usa ese ConfigMap en un Pod llamado `config-pod` con la imagen `busybox` para que pueda leer `APP_MODE` como variable de entorno.  

8. **Montar un Secret en un Pod**  
   - Crea un Secret llamado `db-secret` en el namespace `secrets-test` con:  
     - `DB_USER=admin`  
     - `DB_PASSWORD=SuperSecret123`  
   - Crea un Pod llamado `db-pod` con la imagen `mysql:5.7` que use las credenciales del Secret como variables de entorno.  

---

### 🔹 **Ejercicios sobre Storage y Volumes**
9. **Crear un Pod con un PersistentVolumeClaim**  
   - Define un PersistentVolume (PV) de 1Gi en `storage-test`.  
   - Crea un PersistentVolumeClaim (PVC) que solicite 500Mi.  
   - Usa el PVC en un Pod `storage-pod` con la imagen `busybox`, montando el volumen en `/data`.  

10. **Probar EmptyDir para Datos Temporales**  
    - Crea un Pod `temp-storage` en `ephemeral-test` con dos contenedores (`app1` y `app2`).  
    - Ambos deben compartir un volumen `emptyDir` en `/tmp`.  
    - Usa `kubectl exec` para verificar que ambos contenedores pueden escribir en `/tmp`.  

---

### 🔹 **Ejercicios sobre RBAC y Seguridad**
11. **Configurar RBAC para un Usuario**  
    - Crea un ServiceAccount llamado `viewer-user` en `rbac-test`.  
    - Dale permisos de solo lectura a Pods en ese namespace mediante un Role y RoleBinding.  

12. **Restringir un Pod con SecurityContext**  
    - Crea un Pod `secure-pod` en `security-test` con `nginx`.  
    - Configura `runAsUser: 1001`, `readOnlyRootFilesystem: true` y limita los recursos de CPU y memoria.  

---

Estos ejercicios cubren lo esencial para la certificación de Kubernetes. Si necesitas más en algún área específica, dime y te preparo más retos. 🚀