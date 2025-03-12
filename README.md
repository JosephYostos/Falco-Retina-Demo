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

## Installing Retina
