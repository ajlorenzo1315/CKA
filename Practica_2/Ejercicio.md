 
# 2025/03/11

[https://kubernetes.io/docs/tutorials/hello-minikube/](https://kubernetes.io/docs/tutorials/hello-minikube/)

https://kubernetes.io/docs/reference/kubectl/



# Open the Dashboard
Open the Kubernetes dashboard. You can do this two different ways:

Launch a browser
URL copy and paste
Open a new terminal, and run:

# Start a new terminal, and leave this running.

```bash
minikube dashboard



http://10.88.0.3:44555/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

# Create a Deployment
A Kubernetes Pod is a group of one or more Containers, tied together for the purposes of administration and networking. The Pod in this tutorial has only one Container. A Kubernetes Deployment checks on the health of your Pod and restarts the Pod's Container if it terminates. Deployments are the recommended way to manage the creation and scaling of Pods.

1 Use the kubectl create command to create a Deployment that manages a Pod. The Pod runs a Container based on the provided Docker image.

```bash
# Use the kubectl create command to create a Deployment that manages a Pod. The Pod runs a Container based on the provided Docker image. Run a test container image that includes a webserver
ajlorenzo_1315@cloudshell:~$ kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
deployment.apps/hello-node created
# View the Deployment: (It may take some time for the pod to become available. If you see "0/1", try again in a few seconds.)
ajlorenzo_1315@cloudshell:~$ kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   1/1     1            1           7s

# View the Pod:
ajlorenzo_1315@cloudshell:~$ kubectl get pods
NAME                         READY   STATUS    RESTARTS      AGE
hello-node-c74958b5d-bntfq   1/1     Running   0             13s
multi-container-pod          2/2     Running   1 (12m ago)   72m
nginx-rc-6q2sz               1/1     Running   0             45m
nginx-rc-7f2vk               1/1     Running   0             45m
nginx-rc-nq4xd               1/1     Running   0             45m

# View cluster events:
ajlorenzo_1315@cloudshell:~$ kubectl get events
LAST SEEN   TYPE     REASON              OBJECT                            MESSAGE
87s         Normal   Scheduled           pod/hello-node-c74958b5d-bntfq    Successfully assigned default/hello-node-c74958b5d-bntfq to minikube
86s         Normal   Pulling             pod/hello-node-c74958b5d-bntfq    Pulling image "registry.k8s.io/e2e-test-images/agnhost:2.39"
83s         Normal   Pulled              pod/hello-node-c74958b5d-bntfq    Successfully pulled image "registry.k8s.io/e2e-test-images/agnhost:2.39" in 2.881s (2.881s including waiting). Image size: 126872991 bytes.
83s         Normal   Created             pod/hello-node-c74958b5d-bntfq    Created container: agnhost
83s         Normal   Started             pod/hello-node-c74958b5d-bntfq    Started container agnhost
87s         Normal   SuccessfulCreate    replicaset/hello-node-c74958b5d   Created pod: hello-node-c74958b5d-bntfq
87s         Normal   ScalingReplicaSet   deployment/hello-node             Scaled up replica set hello-node-c74958b5d from 0 to 1

# View the kubectl configuration:

ajlorenzo_1315@cloudshell:~$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /google/minikube/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 11 Mar 2025 15:22:10 UTC
        provider: minikube.sigs.k8s.io
        version: v1.35.0
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Tue, 11 Mar 2025 15:22:10 UTC
        provider: minikube.sigs.k8s.io
        version: v1.35.0
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /google/minikube/.minikube/profiles/minikube/client.crt
    client-key: /google/minikube/.minikube/profiles/minikube/client.key


# get logs

ajlorenzo_1315@cloudshell:~$ kubectl logs hello-node-5f76cf6ccf-br9b5
error: error from server (NotFound): pods "hello-node-5f76cf6ccf-br9b5" not found in namespace "default"
ajlorenzo_1315@cloudshell:~$ kubectl get pods
NAME                         READY   STATUS    RESTARTS      AGE
hello-node-c74958b5d-bntfq   1/1     Running   0             2m43s
multi-container-pod          2/2     Running   1 (14m ago)   74m
nginx-rc-6q2sz               1/1     Running   0             48m
nginx-rc-7f2vk               1/1     Running   0             48m
nginx-rc-nq4xd               1/1     Running   0             48m
ajlorenzo_1315@cloudshell:~$ kubectl logs hello-node
error: error from server (NotFound): pods "hello-node" not found in namespace "default"
ajlorenzo_1315@cloudshell:~$ kubectl logs hello-node-c74958b5d-bntfq
I0311 16:59:16.912179       1 log.go:195] Started HTTP server on port 8080
I0311 16:59:16.914138       1 log.go:195] Started UDP server on port  8081

```

# Create a Service


```bash

# Expose the Pod to the public internet using the kubectl expose command: 
# The --type=LoadBalancer flag indicates that you want to expose your Service outside of the cluster.

#The application code inside the test image only listens on TCP port 8080. If you used kubectl expose to expose a different port, clients could not connect to that other port.

ajlorenzo_1315@cloudshell:~$ kubectl expose deployment hello-node --type=LoadBalancer --port=8080
service/hello-node exposed

# View the Service you created: in minikube, the LoadBalancer type makes the Service accessible through the minikube service command.
ajlorenzo_1315@cloudshell:~$ kubectl get services
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.99.136.211   <pending>     8080:32708/TCP   7s
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          100m
nginx-rc     ClusterIP      10.97.14.40     <none>        80/TCP           29m

# opens up a browser window that serves your app and shows the app's response.

ajlorenzo_1315@cloudshell:~$ minikube service hello-node
|-----------|------------|-------------|---------------------------|
| NAMESPACE |    NAME    | TARGET PORT |            URL            |
|-----------|------------|-------------|---------------------------|
| default   | hello-node |        8080 | http://192.168.49.2:32708 |
|-----------|------------|-------------|---------------------------|
* Opening service default/hello-node in default browser...
  http://192.168.49.2:32708

```

# Enable addons


```bash

ajlorenzo_1315@cloudshell:~$ minikube addons list
|-----------------------------|----------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE  |    STATUS    |           MAINTAINER           |
|-----------------------------|----------|--------------|--------------------------------|
| ambassador                  | minikube | disabled     | 3rd party (Ambassador)         |
| amd-gpu-device-plugin       | minikube | disabled     | 3rd party (AMD)                |
| auto-pause                  | minikube | disabled     | minikube                       |
| cloud-spanner               | minikube | disabled     | Google                         |
| csi-hostpath-driver         | minikube | disabled     | Kubernetes                     |
| dashboard                   | minikube | enabled ✅   | Kubernetes                     |
| default-storageclass        | minikube | enabled ✅   | Kubernetes                     |
| efk                         | minikube | disabled     | 3rd party (Elastic)            |
| freshpod                    | minikube | disabled     | Google                         |
| gcp-auth                    | minikube | disabled     | Google                         |
| gvisor                      | minikube | disabled     | minikube                       |
| headlamp                    | minikube | disabled     | 3rd party (kinvolk.io)         |
| inaccel                     | minikube | disabled     | 3rd party (InAccel             |
|                             |          |              | [info@inaccel.com])            |
| ingress                     | minikube | disabled     | Kubernetes                     |
| ingress-dns                 | minikube | disabled     | minikube                       |
| inspektor-gadget            | minikube | disabled     | 3rd party                      |
|                             |          |              | (inspektor-gadget.io)          |
| istio                       | minikube | disabled     | 3rd party (Istio)              |
| istio-provisioner           | minikube | disabled     | 3rd party (Istio)              |
| kong                        | minikube | disabled     | 3rd party (Kong HQ)            |
| kubeflow                    | minikube | disabled     | 3rd party                      |
| kubevirt                    | minikube | disabled     | 3rd party (KubeVirt)           |
| logviewer                   | minikube | disabled     | 3rd party (unknown)            |
| metallb                     | minikube | disabled     | 3rd party (MetalLB)            |
| metrics-server              | minikube | disabled     | Kubernetes                     |
| nvidia-device-plugin        | minikube | disabled     | 3rd party (NVIDIA)             |
| nvidia-driver-installer     | minikube | disabled     | 3rd party (NVIDIA)             |
| nvidia-gpu-device-plugin    | minikube | disabled     | 3rd party (NVIDIA)             |
| olm                         | minikube | disabled     | 3rd party (Operator Framework) |
| pod-security-policy         | minikube | disabled     | 3rd party (unknown)            |
| portainer                   | minikube | disabled     | 3rd party (Portainer.io)       |
| registry                    | minikube | disabled     | minikube                       |
| registry-aliases            | minikube | disabled     | 3rd party (unknown)            |
| registry-creds              | minikube | disabled     | 3rd party (UPMC Enterprises)   |
| storage-provisioner         | minikube | enabled ✅   | minikube                       |
| storage-provisioner-gluster | minikube | disabled     | 3rd party (Gluster)            |
| storage-provisioner-rancher | minikube | disabled     | 3rd party (Rancher)            |
| volcano                     | minikube | disabled     | third-party (volcano)          |
| volumesnapshots             | minikube | disabled     | Kubernetes                     |
| yakd                        | minikube | disabled     | 3rd party (marcnuri.com)       |
|-----------------------------|----------|--------------|--------------------------------|
ajlorenzo_1315@cloudshell:~$ minikube addons enable metrics-server
* metrics-server is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
  - Using image registry.k8s.io/metrics-server/metrics-server:v0.7.2
* The 'metrics-server' addon is enabled
ajlorenzo_1315@cloudshell:~$ kubectl get pod,svc -n kube-system
NAME                                   READY   STATUS    RESTARTS       AGE
pod/coredns-668d6bf9bc-7p4xc           1/1     Running   0              103m
pod/etcd-minikube                      1/1     Running   0              103m
pod/kube-apiserver-minikube            1/1     Running   0              103m
pod/kube-controller-manager-minikube   1/1     Running   0              103m
pod/kube-proxy-6k7qr                   1/1     Running   0              103m
pod/kube-scheduler-minikube            1/1     Running   0              103m
pod/metrics-server-7fbb699795-764hz    0/1     Running   0              6s
pod/storage-provisioner                1/1     Running   1 (102m ago)   103m

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns         ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   103m
service/metrics-server   ClusterIP   10.110.182.218   <none>        443/TCP                  6s
ajlorenzo_1315@cloudshell:~$ minikube addons enable metrics-server
* metrics-server is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
  - Using image registry.k8s.io/metrics-server/metrics-server:v0.7.2
* The 'metrics-server' addon is enabled

# If you see the following message, wait, and try again:error: Metrics API not available


ajlorenzo_1315@cloudshell:~$ kubectl get pod,svc -n kube-system
NAME                                   READY   STATUS    RESTARTS       AGE
pod/coredns-668d6bf9bc-7p4xc           1/1     Running   0              103m
pod/etcd-minikube                      1/1     Running   0              103m
pod/kube-apiserver-minikube            1/1     Running   0              103m
pod/kube-controller-manager-minikube   1/1     Running   0              103m
pod/kube-proxy-6k7qr                   1/1     Running   0              103m
pod/kube-scheduler-minikube            1/1     Running   0              103m
pod/metrics-server-7fbb699795-764hz    0/1     Running   0              39s
pod/storage-provisioner                1/1     Running   1 (103m ago)   103m

NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns         ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   103m
service/metrics-server   ClusterIP   10.110.182.218   <none>        443/TCP                  39s
ajlorenzo_1315@cloudshell:~$ kubectl top pods
NAME                         CPU(cores)   MEMORY(bytes)   
hello-node-c74958b5d-bntfq   1m           5Mi             
multi-container-pod          0m           3Mi             
nginx-rc-6q2sz               0m           3Mi             
nginx-rc-7f2vk               0m           3Mi             
nginx-rc-nq4xd               0m           3Mi             
ajlorenzo_1315@cloudshell:~$ minikube addons disable metrics-server
* "The 'metrics-server' addon is disabled

```

# Clean up

```bash 
ajlorenzo_1315@cloudshell:~$ minikube addons disable metrics-server
* "The 'metrics-server' addon is disabled
ajlorenzo_1315@cloudshell:~$ kubectl delete service hello-node
kubectl delete deployment hello-node
service "hello-node" deleted
deployment.apps "hello-node" deleted
ajlorenzo_1315@cloudshell:~$ kubectl get  pods
NAME                  READY   STATUS    RESTARTS      AGE
multi-container-pod   2/2     Running   1 (21m ago)   81m
nginx-rc-6q2sz        1/1     Running   0             54m
nginx-rc-7f2vk        1/1     Running   0             54m
nginx-rc-nq4xd        1/1     Running   0             54m

```