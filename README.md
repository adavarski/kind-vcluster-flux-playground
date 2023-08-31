## (WIP) kind + vcluster + flux multi-tenancy PoC
[vcluster](https://www.vcluster.com/) + [flux](https://fluxcd.io/) multi-tenancy  PoC

vcluster - Create fully functional virtual Kubernetes clusters - Each vcluster runs inside a namespace of the underlying k8s cluster. It's cheaper than creating separate full-blown clusters and it offers better multi-tenancy and isolation than regular namespaces.

Architecture:

<img src="pictures/vcluster-architecture.svg?raw=true" width="1000">

Why Virtual Kubernetes Clusters?

<img src="pictures/vcluster-comparison.png?raw=true" width="600">

### Requirenments
- Linux laptop/workstation
- Go

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

``` 

### Configure vcluster contexts

```bash
make vctx

Example Output:

$ make vctx
Switched to context "kind-host-cluster".
/home/davar/Downloads/kind-vcluster-flux-playground/bin/vcluster connect vcluster-a -n vcluster-a
info   Waiting for vcluster to come up...
info   Using vcluster vcluster-a load balancer endpoint: 172.18.0.210
done √ Switched active kube context to vcluster_vcluster-a_vcluster-a_kind-host-cluster
- Use `vcluster disconnect` to return to your previous kube context
- Use `kubectl get namespaces` to access the vcluster
/home/davar/Downloads/kind-vcluster-flux-playground/bin/vcluster connect vcluster-b -n vcluster-b
info   Using vcluster vcluster-b load balancer endpoint: 172.18.0.211
done √ Switched active kube context to vcluster_vcluster-b_vcluster-b_kind-host-cluster
- Use `vcluster disconnect` to return to your previous kube context
- Use `kubectl get namespaces` to access the vcluster

$ ./bin/vcluster connect vcluster-a -n vcluster-a
info   Using vcluster vcluster-a load balancer endpoint: 172.18.0.210
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
NAMESPACE     NAME                       READY   STATUS             RESTARTS        AGE
default       nginx-748c667d99-84cl7     1/1     Running            0               5m14s
kube-system   coredns-56bfc489cf-ggh5b   0/1     CrashLoopBackOff   6 (3m19s ago)   9m28s

$ kubectl get events -n kube-system
LAST SEEN   TYPE      REASON              OBJECT                          MESSAGE
21m         Normal    ApplyingManifest    addon/rolebindings              Applying manifest at "/data/server/manifests/rolebindings.yaml"
21m         Normal    AppliedManifest     addon/rolebindings              Applied manifest at "/data/server/manifests/rolebindings.yaml"
21m         Normal    ScalingReplicaSet   deployment/coredns              Scaled up replica set coredns-56bfc489cf to 1
21m         Normal    SuccessfulCreate    replicaset/coredns-56bfc489cf   Created pod: coredns-56bfc489cf-ggh5b
21m         Normal    Scheduled           pod/coredns-56bfc489cf-ggh5b    Successfully assigned kube-system/coredns-56bfc489cf-ggh5b to host-cluster-worker2
21m         Normal    Pulling             pod/coredns-56bfc489cf-ggh5b    Pulling image "coredns/coredns"
20m         Normal    Pulled              pod/coredns-56bfc489cf-ggh5b    Successfully pulled image "coredns/coredns" in 2.841619166s (9.026249629s including waiting)
19m         Normal    Pulled              pod/coredns-56bfc489cf-ggh5b    Container image "coredns/coredns" already present on machine
19m         Normal    Created             pod/coredns-56bfc489cf-ggh5b    Created container coredns
19m         Normal    Started             pod/coredns-56bfc489cf-ggh5b    Started container coredns
57s         Warning   BackOff             pod/coredns-56bfc489cf-ggh5b    Back-off restarting failed container coredns in pod coredns-56bfc489cf-ggh5b_vcluster-a(3dbd12b8-cb1c-4525-b4b1-db01095699d0)
./bin/vcluster disconnect

$ ./bin/vcluster connect vcluster-b -n vcluster-b
info   Using vcluster vcluster-b load balancer endpoint: 172.18.0.211
done √ Switched active kube context to vcluster_vcluster-b_vcluster-b_kind-host-cluster
- Use `vcluster disconnect` to return to your previous kube context
- Use `kubectl get namespaces` to access the vcluster
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   6m10s
kube-system       Active   6m10s
kube-public       Active   6m10s
kube-node-lease   Active   6m10s
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

