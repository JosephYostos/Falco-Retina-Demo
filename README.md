# Falco-Retina-Demo
Quick Demo of Falco Talon + Retina Response Actions

## Setting up the Kubernetes cluster

On an ```Ubuntu 20.04``` EC2 instance, we will install the containerD runtime components:
```
curl https://raw.githubusercontent.com/xxradar/install_k8s_ubuntu/main/setup_latest.sh | bash
```

By default, we will face ```Taints``` that affect the scheduling of our pods:
```
kubectl get nodes
```
Simply remove the taints on our ```control plane``` node.
```
kubectl taint node $(kubectl get nodes --selector='node-role.kubernetes.io/control-plane' -o jsonpath='{.items[0].metadata.name}') node-role.kubernetes.io/control-plane:NoSchedule-
```

We are going to install ```Cilium``` as our CNI of choice to ensure networking on the cluster
```
curl https://raw.githubusercontent.com/xxradar/k8s-calico-oss-install-containerd/refs/heads/main/cilium_install.sh | bash
```

## Installing Falco + Talon
```
helm search repo falcosecurity/falco-talon
```



## Installing Retina
Basic Mode (with Capture Support) <br/>
```https://retina.sh/docs/Installation/Setup#basic-mode-with-capture-support```

```
VERSION=$( curl -sL https://api.github.com/repos/microsoft/retina/releases/latest | jq -r .name)
helm upgrade --install retina oci://ghcr.io/microsoft/retina/charts/retina \
    --version $VERSION \
    --namespace kube-system \
    --set image.tag=$VERSION \
    --set operator.tag=$VERSION \
    --set logLevel=info \
    --set image.pullPolicy=Always \
    --set logLevel=info \
    --set os.windows=true \
    --set operator.enabled=true \
    --set operator.enableRetinaEndpoint=true \
    --skip-crds \
    --set enabledPlugin_linux="\[dropreason\,packetforward\,linuxutil\,dns\,packetparser\]"
```
