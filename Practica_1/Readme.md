Primeros pasos 

usando una sistema virtual  [https://labs.play-with-k8s.com/](https://labs.play-with-k8s.com/)

# Iniciamos nodo

```bash 

kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16

es utilizado para inicializar un clúster de Kubernetes en un nodo maestro utilizando kubeadm. Analicemos cada parte del comando:

Explicación de los parámetros
kubeadm init

Es el comando principal para inicializar un clúster de Kubernetes en un nodo maestro.
--apiserver-advertise-address $(hostname -i)

Define la dirección IP en la que el API Server de Kubernetes va a estar escuchando las conexiones desde otros nodos.
$(hostname -i) obtiene la dirección IP del hostname del nodo.
Esto es útil si el servidor tiene múltiples interfaces de red y deseas especificar cuál debe ser utilizada.
--pod-network-cidr 10.5.0.0/16

Define el rango de direcciones IP para los Pods en el clúster.
Esto es importante porque algunas redes de Kubernetes (CNI como Flannel o Calico) requieren esta configuración para saber cómo enrutar los paquetes dentro del clúster.
En este caso, los Pods utilizarán direcciones IP dentro del rango 10.5.0.0/16.
Pasos después de ejecutar el comando
Una vez ejecutado este comando, deberás:

Aplicar una CNI (Container Network Interface), como Calico o Flannel, para que la red del clúster funcione correctamente.
Unir nodos trabajadores al clúster usando el token que kubeadm genera al finalizar la inicialización.
Configurar kubectl en el nodo maestro para gestionar el clúster.

```
### Error que me sale al usarlo 

```bash
kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
W0310 18:16:44.195962    1958 initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future. Automatically prepending scheme "unix" to the "criSocket" with value "/run/docker/containerd/containerd.sock". Please update your configuration!
I0310 18:16:44.521691    1958 version.go:256] remote version is much newer: v1.32.2; falling back to: stable-1.27
[init] Using Kubernetes version: v1.27.16
[preflight] Running pre-flight checks
        [WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
[preflight] The system verification failed. Printing the output from the verification:
KERNEL_VERSION: 4.4.0-210-generic
OS: Linux
CGROUPS_CPU: enabled
CGROUPS_CPUACCT: enabled
CGROUPS_CPUSET: enabled
CGROUPS_DEVICES: enabled
CGROUPS_FREEZER: enabled
CGROUPS_MEMORY: enabled
CGROUPS_PIDS: enabled
CGROUPS_HUGETLB: enabled
CGROUPS_BLKIO: enabled
        [WARNING SystemVerification]: failed to parse kernel config: unable to load kernel module: "configs", output: "", err: exit status 1
        [WARNING FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
W0310 18:16:44.972225    1958 images.go:80] could not find officially supported version of etcd for Kubernetes v1.27.16, falling back to the nearest etcd version (3.5.7-0)
W0310 18:16:52.494714    1958 checks.go:835] detected that the sandbox image "registry.k8s.io/pause:3.6" of the container runtime is inconsistent with that used by kubeadm. It is recommended that using "registry.k8s.io/pause:3.9" as the CRI sandbox image.
        [WARNING ImagePull]: failed to pull image registry.k8s.io/kube-apiserver:v1.27.16: output: E0310 18:16:47.048579    2082 remote_image.go:171] "PullImage from image service failed" err="rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/kube-apiserver:v1.27.16\": failed to extract layer sha256:af5aa97ebe6ce1604747ec1e21af7136ded391bcabe4acef882e718a87c86bcc: write /var/lib/docker/containerd/daemon/io.containerd.snapshotter.v1.overlayfs/snapshots/45/fs/etc/passwd: no space left on device: unknown" image="registry.k8s.io/kube-apiserver:v1.27.16"
time="2025-03-10T18:16:47Z" level=fatal msg="pulling image: rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/kube-apiserver:v1.27.16\": failed to extract layer sha256:af5aa97ebe6ce1604747ec1e21af7136ded391bcabe4acef882e718a87c86bcc: write /var/lib/docker/containerd/daemon/io.containerd.snapshotter.v1.overlayfs/snapshots/45/fs/etc/passwd: no space left on device: unknown"
, error: exit status 1
        [WARNING ImagePull]: failed to pull image registry.k8s.io/kube-controller-manager:v1.27.16: output: E0310 18:16:48.724073    2134 remote_image.go:171] "PullImage from image service failed" err="rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/kube-controller-manager:v1.27.16\": write /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/ingest/6cfddf258058374addd0b41938402cd3ef03827dcfe1055de336fc9d97e8d33f/ref: no space left on device" image="registry.k8s.io/kube-controller-manager:v1.27.16"
time="2025-03-10T18:16:48Z" level=fatal msg="pulling image: rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/kube-controller-manager:v1.27.16\": write /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/ingest/6cfddf258058374addd0b41938402cd3ef03827dcfe1055de336fc9d97e8d33f/ref: no space left on device"
, error: exit status 1
        [WARNING ImagePull]: failed to pull image registry.k8s.io/kube-scheduler:v1.27.16: output: E0310 18:16:51.279374    2185 remote_image.go:171] "PullImage from image service failed" err="rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/kube-scheduler:v1.27.16\": failed to extract layer sha256:06d84c7e474c1b4311890a004c0880577d0fb8d3b89639b7541765289ca36355: write /var/lib/docker/containerd/daemon/io.containerd.snapshotter.v1.overlayfs/snapshots/62/fs/usr/local/bin/kube-scheduler: no space left on device: unknown" image="registry.k8s.io/kube-scheduler:v1.27.16"
time="2025-03-10T18:16:51Z" level=fatal msg="pulling image: rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/kube-scheduler:v1.27.16\": failed to extract layer sha256:06d84c7e474c1b4311890a004c0880577d0fb8d3b89639b7541765289ca36355: write /var/lib/docker/containerd/daemon/io.containerd.snapshotter.v1.overlayfs/snapshots/62/fs/usr/local/bin/kube-scheduler: no space left on device: unknown"
, error: exit status 1
        [WARNING ImagePull]: failed to pull image registry.k8s.io/kube-proxy:v1.27.16: output: E0310 18:16:52.441166    2249 remote_image.go:171] "PullImage from image service failed" err="rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/kube-proxy:v1.27.16\": write /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/ingest/8d016fadf5bce09a5248c41603f3e9dadd8dd400e3a8944c009e6a582e79b6a3/ref: no space left on device" image="registry.k8s.io/kube-proxy:v1.27.16"
time="2025-03-10T18:16:52Z" level=fatal msg="pulling image: rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/kube-proxy:v1.27.16\": write /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/ingest/8d016fadf5bce09a5248c41603f3e9dadd8dd400e3a8944c009e6a582e79b6a3/ref: no space left on device"
, error: exit status 1
        [WARNING ImagePull]: failed to pull image registry.k8s.io/etcd:3.5.7-0: output: E0310 18:16:55.016940    2358 remote_image.go:171] "PullImage from image service failed" err="rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/etcd:3.5.7-0\": write /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/ingest/82adaccf0db06e6e4b62ce497839569eaa8bf3ebd5719b5c08da42a289fcbfb1/total: no space left on device" image="registry.k8s.io/etcd:3.5.7-0"
time="2025-03-10T18:16:55Z" level=fatal msg="pulling image: rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/etcd:3.5.7-0\": write /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/ingest/82adaccf0db06e6e4b62ce497839569eaa8bf3ebd5719b5c08da42a289fcbfb1/total: no space left on device"
, error: exit status 1
        [WARNING ImagePull]: failed to pull image registry.k8s.io/coredns/coredns:v1.10.1: output: E0310 18:16:56.046116    2409 remote_image.go:171] "PullImage from image service failed" err="rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/coredns/coredns:v1.10.1\": failed commit on ref \"index-sha256:a0ead06651cf580044aeb0a0feba63591858fb2e43ade8c9dea45a6a89ae7e5e\": rename /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/ingest/5a672d24a1679f746143349751ec985998f7f768a4511978ee7f439f43f50d8b/data /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/blobs/sha256/a0ead06651cf580044aeb0a0feba63591858fb2e43ade8c9dea45a6a89ae7e5e: no space left on device" image="registry.k8s.io/coredns/coredns:v1.10.1"
time="2025-03-10T18:16:56Z" level=fatal msg="pulling image: rpc error: code = Unknown desc = failed to pull and unpack image \"registry.k8s.io/coredns/coredns:v1.10.1\": failed commit on ref \"index-sha256:a0ead06651cf580044aeb0a0feba63591858fb2e43ade8c9dea45a6a89ae7e5e\": rename /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/ingest/5a672d24a1679f746143349751ec985998f7f768a4511978ee7f439f43f50d8b/data /var/lib/docker/containerd/daemon/io.containerd.content.v1.content/blobs/sha256/a0ead06651cf580044aeb0a0feba63591858fb2e43ade8c9dea45a6a89ae7e5e: no space left on device"
, error: exit status 1
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
error execution phase certs/ca: failure while saving ca certificate and key: couldn't write key: unable to write private key to file /etc/kubernetes/pki/ca.key: write /etc/kubernetes/pki/ca.key: no space left on device
To see the stack trace of this error execute with --v=5 or higher 
```
## solucion reiniciar el sitema para que me diese otra ip no se que paso 

```bash
[node1 ~]$ kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
Initializing machine ID from random generator.
W0310 18:25:42.353358     438 initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future. Automatically prepending scheme "unix" to the "criSocket" with value "/run/docker/containerd/containerd.sock". Please update your configuration!
I0310 18:25:42.859093     438 version.go:256] remote version is much newer: v1.32.2; falling back to: stable-1.27
[init] Using Kubernetes version: v1.27.16
[preflight] Running pre-flight checks
[preflight] The system verification failed. Printing the output from the verification:
KERNEL_VERSION: 4.4.0-210-generic
OS: Linux
CGROUPS_CPU: enabled
CGROUPS_CPUACCT: enabled
CGROUPS_CPUSET: enabled
CGROUPS_DEVICES: enabled
CGROUPS_FREEZER: enabled
CGROUPS_MEMORY: enabled
CGROUPS_PIDS: enabled
CGROUPS_HUGETLB: enabled
CGROUPS_BLKIO: enabled
        [WARNING SystemVerification]: failed to parse kernel config: unable to load kernel module: "configs", output: "", err: exit status 1
        [WARNING FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
W0310 18:25:43.680019     438 images.go:80] could not find officially supported version of etcd for Kubernetes v1.27.16, falling back to the nearest etcd version (3.5.7-0)
W0310 18:25:57.480130     438 checks.go:835] detected that the sandbox image "registry.k8s.io/pause:3.6" of the container runtime is inconsistent with that used by kubeadm. It is recommended that using "registry.k8s.io/pause:3.9" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local node1] and IPs [10.96.0.1 192.168.0.13]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost node1] and IPs [192.168.0.13 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost node1] and IPs [192.168.0.13 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
W0310 18:26:18.066805     438 images.go:80] could not find officially supported version of etcd for Kubernetes v1.27.16, falling back to the nearest etcd version (3.5.7-0)
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 12.013509 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node node1 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node node1 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: xa2bbq.eexk6ln2ywxzbind
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.13:6443 --token xa2bbq.eexk6ln2ywxzbind \
        --discovery-token-ca-cert-hash sha256:daf5859dde852c2afdaa36bba51ac488c7eba65ea59f3c5ed0f222072dd808bb 
Waiting for api server to startup
Warning: resource daemonsets/kube-proxy is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
daemonset.apps/kube-proxy configured
No resources found
[node1 ~]$ kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
Initializing machine ID from random generator.
W0310 18:25:42.353358     438 initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future. Automatically prepending scheme "unix" to the "criSocket" with value "/run/docker/containerd/containerd.sock". Please update your configuration!
I0310 18:25:42.859093     438 version.go:256] remote version is much newer: v1.32.2; falling back to: stable-1.27
[init] Using Kubernetes version: v1.27.16
[preflight] Running pre-flight checks
[preflight] The system verification failed. Printing the output from the verification:
KERNEL_VERSION: 4.4.0-210-generic
OS: Linux
CGROUPS_CPU: enabled
CGROUPS_CPUACCT: enabled
CGROUPS_CPUSET: enabled
CGROUPS_DEVICES: enabled
CGROUPS_FREEZER: enabled
CGROUPS_MEMORY: enabled
CGROUPS_PIDS: enabled
CGROUPS_HUGETLB: enabled
CGROUPS_BLKIO: enabled
        [WARNING SystemVerification]: failed to parse kernel config: unable to load kernel module: "configs", output: "", err: exit status 1
        [WARNING FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
W0310 18:25:43.680019     438 images.go:80] could not find officially supported version of etcd for Kubernetes v1.27.16, falling back to the nearest etcd version (3.5.7-0)
W0310 18:25:57.480130     438 checks.go:835] detected that the sandbox image "registry.k8s.io/pause:3.6" of the container runtime is inconsistent with that used by kubeadm. It is recommended that using "registry.k8s.io/pause:3.9" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local node1] and IPs [10.96.0.1 192.168.0.13]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost node1] and IPs [192.168.0.13 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost node1] and IPs [192.168.0.13 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
W0310 18:26:18.066805     438 images.go:80] could not find officially supported version of etcd for Kubernetes v1.27.16, falling back to the nearest etcd version (3.5.7-0)
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 12.013509 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node node1 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node node1 as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: xa2bbq.eexk6ln2ywxzbind
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.13:6443 --token xa2bbq.eexk6ln2ywxzbind \
        --discovery-token-ca-cert-hash sha256:daf5859dde852c2afdaa36bba51ac488c7eba65ea59f3c5ed0f222072dd808bb 
Waiting for api server to startup
Warning: resource daemonsets/kube-proxy is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
daemonset.apps/kube-proxy configured
No resources found

```

## En un nuevo nodo 

```bash 
kubeadm join 192.168.0.13:6443 --token xa2bbq.eexk6ln2ywxzbind \
        --discovery-token-ca-cert-hash sha256:daf5859dde852c2afdaa36bba51ac488c7eba65ea59f3c5ed0f222072dd808bb 
```

```bash 
$ kubeadm join 192.168.0.13:6443 --token xa2bbq.eexk6ln2ywxzbind \
>         --discovery-token-ca-cert-hash sha256:daf5859dde852c2afdaa36bba51ac488c7eba65ea59f3c5ed0f222072dd808bb 
Initializing machine ID from random generator.
W0310 18:30:15.611531     441 initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future. Automatically prepending scheme "unix" to the "criSocket" with value "/run/docker/containerd/containerd.sock". Please update your configuration!
[preflight] Running pre-flight checks
[preflight] The system verification failed. Printing the output from the verification:
KERNEL_VERSION: 4.4.0-210-generic
OS: Linux
CGROUPS_CPU: enabled
CGROUPS_CPUACCT: enabled
CGROUPS_CPUSET: enabled
CGROUPS_DEVICES: enabled
CGROUPS_FREEZER: enabled
CGROUPS_MEMORY: enabled
CGROUPS_PIDS: enabled
CGROUPS_HUGETLB: enabled
CGROUPS_BLKIO: enabled
        [WARNING SystemVerification]: failed to parse kernel config: unable to load kernel module: "configs", output: "", err: exit status 1
        [WARNING FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
```


### En el nodo master  configuramos la red 


```bash 
kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml

```

``` bash
[node1 ~]$ kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
configmap/kube-router-cfg created
daemonset.apps/kube-router created
serviceaccount/kube-router created
clusterrole.rbac.authorization.k8s.io/kube-router created
clusterrolebinding.rbac.authorization.k8s.io/kube-router created

```

**Mirar si funciona**

```bash 
[node1 ~]$ kubectl get node
NAME    STATUS     ROLES           AGE     VERSION
node1   NotReady   control-plane   5m48s   v1.27.2
node2   NotReady   <none>          5m3s    v1.27.2
[node1 ~]$ 
[node1 ~]$ kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
configmap/kube-router-cfg created
daemonset.apps/kube-router created
serviceaccount/kube-router created
clusterrole.rbac.authorization.k8s.io/kube-router created
clusterrolebinding.rbac.authorization.k8s.io/kube-router created
[node1 ~]$ kubectl get node
NAME    STATUS     ROLES           AGE     VERSION
node1   NotReady   control-plane   6m26s   v1.27.2
node2   NotReady   <none>          5m41s   v1.27.2

[node1 ~]$ kubectl get pods -A
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-5d78c9869d-2wf7p        1/1     Running   0          11m
kube-system   coredns-5d78c9869d-wgf6b        1/1     Running   0          11m
kube-system   etcd-node1                      1/1     Running   0          9m55s
kube-system   kube-apiserver-node1            1/1     Running   0          11m
kube-system   kube-controller-manager-node1   1/1     Running   0          10m
kube-system   kube-proxy-4977h                1/1     Running   0          10m
kube-system   kube-proxy-hss6q                1/1     Running   0          11m
kube-system   kube-router-4xsl9               1/1     Running   0          5m19s
kube-system   kube-router-9kqpg               1/1     Running   0          5m19s
kube-system   kube-scheduler-node1            1/1     Running   0          9m54s

kubectl get nodes -o wide
NAME    STATUS     ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION      CONTAINER-RUNTIME
node1   NotReady   control-plane   15m   v1.27.2   192.168.0.12   <none>        CentOS Linux 7 (Core)   4.4.0-210-generic   containerd://1.6.21
node2   Ready      <none>          14m   v1.27.2   192.168.0.13   <none>        CentOS Linux 7 (Core)   4.4.0-210-generic   containerd://1.6.21

```
## Pod

```bash

```

---

# alternativa 

usar [google shell](https://shell.cloud.google.com/)

```bash 

# Esto iniciará un clúster de Kubernetes con 2 nodos (1 maestro y 1 worker) en la misma máquina.
minikube start --nodes=2 --driver=docker

# ip 10.88.0.3

minikube node add


```


---

# Otra alternativa 

[https://k0sproject.io/](https://k0sproject.io/)

```bash 

```