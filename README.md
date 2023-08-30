## (WIP) kind + vcluster + flux multi-tenancy PoC
[vcluster](https://www.vcluster.com/) + [flux](https://fluxcd.io/) multi-tenancy  PoC

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


```

### Uninstall

```bash
make uninstall
```
