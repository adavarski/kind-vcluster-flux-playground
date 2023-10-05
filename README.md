## kind + vcluster + flux multi-tenancy PoC
[vcluster](https://www.vcluster.com/) + [flux](https://fluxcd.io/) multi-tenancy  PoC

vcluster - Create fully functional virtual Kubernetes clusters - Each vcluster runs inside a namespace of the underlying k8s cluster. It's cheaper than creating separate full-blown clusters and it offers better multi-tenancy and isolation than regular namespaces.

Architecture:

<img src="pictures/vcluster-architecture.svg?raw=true" width="1000">

Why Virtual Kubernetes Clusters?

<img src="pictures/vcluster-comparison.png?raw=true" width="600">

### Requirenments
- Linux laptop/workstation
- Docker installed
- Go installed

Go install mini-howto
```
$ wget https://go.dev/dl/go1.19.2.linux-amd64.tar.gz
$ sudo tar -C /usr/local -xzf go1.19.2.linux-amd64.tar.gz

file: ~/.profile 
export PATH=$PATH:/usr/local/go/bin

file ~/.bashrc
export GOROOT=/usr/local/go
export PATH=${GOROOT}/bin:${PATH}
export GOPATH=$HOME/go
export PATH=${GOPATH}/bin:${PATH}

$ source ./.profile
$ go version
go version go1.19.2 linux/amd64

```

### Install

```bash
export GITHUB_TOKEN=<your-personal-access-token>
make install

Example Output:

$ make install
"traefik.local" host already exists in /etc/hosts
"tenant-a.traefik.local" host already exists in /etc/hosts
"tenant-b.traefik.local" host already exists in /etc/hosts
/home/davar/kind-vcluster-flux-playground/bin/kind create cluster --name host-cluster --config hack/config/kind.yaml
Creating cluster "host-cluster" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼 
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-host-cluster"
You can now use your cluster with:

kubectl cluster-info --context kind-host-cluster

Have a nice day! 👋
Switched to context "kind-host-cluster".
/home/davar/Downloads/kind-vcluster-flux-playground/bin/flux bootstrap github \
	--owner=adavarski \
	--repository=kind-vcluster-flux-playground \
	--private=false \
	--personal=true \
	--path=clusters/host-cluster
► connecting to github.com
► cloning branch "main" from Git repository "https://github.com/adavarski/kind-vcluster-flux-playground.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ component manifests are up to date
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
✔ public key: ecdsa-sha2-nistp384 AAAAE2VjZHNhLXNoYTItbmlzdHAzODQAAAAIbmlzdHAzODQAAABhBAsezNDmPk2y6kd5+xPJLY/mP1fqxaa698/eGhaLU+1gPM2xykGc6UQwnEqGqZolZ6AnGB2EhtAbWvysOuaA4NebJCTz+UTRNpXINg3j3DCudE3kG1kslONzNJASn9O8Kg==
✔ configured deploy key "flux-system-main-flux-system-./clusters/host-cluster" for "https://github.com/adavarski/kind-vcluster-flux-playground"
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ sync manifests are up to date
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy

$ docker network inspect -f '{{.IPAM.Config}}' kind
[{172.17.0.0/16  172.17.0.1 map[]} {fc00:f853:ccd:e793::/64  fc00:f853:ccd:e793::1 map[]}]
$ grep -ri "172.17.0.2" *
clusters/vcluster-a/vcluster-a_helmrelease.yaml:        metallb.universe.tf/loadBalancerIPs: 172.17.0.210
clusters/vcluster-a/vcluster-a_helmrelease.yaml:      - --tls-san=172.17.0.210
clusters/vcluster-a/vcluster-a_helmrelease.yaml:      - --out-kube-config-server=https://172.17.0.210
clusters/vcluster-b/vcluster-b_helmrelease.yaml:        metallb.universe.tf/loadBalancerIPs: 172.17.0.211
clusters/vcluster-b/vcluster-b_helmrelease.yaml:      - --tls-san=172.17.0.211
clusters/vcluster-b/vcluster-b_helmrelease.yaml:      - --out-kube-config-server=https://172.17.0.211
infrastructure/metallb/config/metallb_ipaddresspool.yaml:    - 172.17.0.200-172.17.0.220
Makefile:TRAEFIK_IP ?= 172.17.0.200
$ cat /etc/hosts|tail -n6
# kind-vcluster-flux-playground
172.17.0.200	traefik.local
# kind-vcluster-flux-playground
172.17.0.200	tenant-a.traefik.local
# kind-vcluster-flux-playground
172.17.0.200	tenant-b.traefik.local

$ curl -sSLo ./bin/vcluster https://github.com/loft-sh/vcluster/releases/download/v0.16.0/vcluster-linux-amd64
$ chmod +x ./bin/vcluster 
$ ./bin/vcluster version

$ ./bin/vcluster list
  
       NAME    | NAMESPACE  | STATUS  | VERSION | CONNECTED |            CREATED             |  AGE   | PRO  
  -------------+------------+---------+---------+-----------+--------------------------------+--------+------
    vcluster-a | vcluster-a | Running | 0.16.0  |           | 2023-10-05 15:47:40 +0300 EEST | 17m21s |      
    vcluster-b | vcluster-b | Running | 0.16.0  |           | 2023-10-05 15:47:40 +0300 EEST | 17m21s |      
  
``` 

### Configure vcluster contexts

```bash
make vctx

Example Output:

$ make vctx
Switched to context "kind-host-cluster".
/home/davar/Downloads/kind-vcluster-flux-playground/bin/vcluster connect vcluster-a -n vcluster-a
info   Using vcluster vcluster-a load balancer endpoint: 172.17.0.210
done √ Switched active kube context to vcluster_vcluster-a_vcluster-a_kind-host-cluster
- Use `vcluster disconnect` to return to your previous kube context
- Use `kubectl get namespaces` to access the vcluster
/home/davar/Downloads/kind-vcluster-flux-playground/bin/vcluster connect vcluster-b -n vcluster-b
info   Using vcluster vcluster-b load balancer endpoint: 172.17.0.211
done √ Switched active kube context to vcluster_vcluster-b_vcluster-b_kind-host-cluster
- Use `vcluster disconnect` to return to your previous kube context
- Use `kubectl get namespaces` to access the vcluster


$ ./bin/vcluster connect vcluster-a -n vcluster-a
info   Using vcluster vcluster-a load balancer endpoint: 172.17.0.210
done √ Switched active kube context to vcluster_vcluster-a_vcluster-a_kind-host-cluster
- Use `vcluster disconnect` to return to your previous kube context
- Use `kubectl get namespaces` to access the vcluster

$ kubectl get namespaces
NAME              STATUS   AGE
kube-system       Active   4m48s
default           Active   4m48s
kube-public       Active   4m48s
kube-node-lease   Active   4m48s

$ kubectl get po --all-namespaces
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
kube-system   coredns-68559449b6-cltc8   1/1     Running   0          16m
default       nginx-77b4fdf86c-7lts5     1/1     Running   0          13m

./bin/vcluster disconnect

$ ./bin/vcluster connect vcluster-b -n vcluster-b
info   Using vcluster vcluster-b load balancer endpoint: 172.17.0.211
done √ Switched active kube context to vcluster_vcluster-b_vcluster-b_kind-host-cluster
- Use `vcluster disconnect` to return to your previous kube context
- Use `kubectl get namespaces` to access the vcluster

$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   6m10s
kube-system       Active   6m10s
kube-public       Active   6m10s
kube-node-lease   Active   6m10s

$ kubectl get po --all-namespaces
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
kube-system   coredns-68559449b6-6m28q   1/1     Running   0          17m
default       nginx-77b4fdf86c-9hrzh     1/1     Running   0          14m

./bin/vcluster disconnect

```

### Uninstall

```bash
make uninstall

Example Output:

$ make uninstall
Switched to context "kind-host-cluster".
/home/davar/kind-vcluster-flux-playground/bin/vcluster delete vcluster-a -n vcluster-a
info   Stopping docker proxy...
info   Delete vcluster vcluster-a...
done √ Successfully deleted virtual cluster vcluster-a in namespace vcluster-a
done √ Successfully deleted virtual cluster pvc data-vcluster-a-0 in namespace vcluster-a
/home/davar/Downloads/kind-vcluster-flux-playground/bin/vcluster delete vcluster-b -n vcluster-b
info   Stopping docker proxy...
info   Delete vcluster vcluster-b...
done √ Successfully deleted virtual cluster vcluster-b in namespace vcluster-b
done √ Successfully deleted virtual cluster pvc data-vcluster-b-0 in namespace vcluster-b
GOBIN=/home/davar/Downloads/kind-vcluster-flux-playground/bin go install sigs.k8s.io/kind@v0.20.0
/home/davar/Downloads/kind-vcluster-flux-playground/bin/kind delete cluster --name host-cluster
Deleting cluster "host-cluster" ...
Deleted nodes: ["host-cluster-worker2" "host-cluster-worker" "host-cluster-control-plane"]

```

TODO: fix coredns for vcusters : https://github.com/loft-sh/vcluster/issues/1151 & https://github.com/loft-sh/vcluster/pull/1152

REF: https://github.com/loft-sh/vcluster

[Credits](https://github.com/mmontes11/vcluster-poc) 

