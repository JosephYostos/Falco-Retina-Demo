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
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
```

Check to see that there is a chart for Falco Talon
```
helm search repo falcosecurity/falco-talon
```

Install Falco without the Talon component. <br/>
Simply configure the falcosidekick component to send events to the future Talon deployment via webhook.
```
helm install falco falcosecurity/falco --namespace falco \
  --create-namespace \
  --set tty=true \
  --set falcosidekick.enabled=true \
  --set falcosidekick.config.talon.address=http://falco-talon:2803
```

### Install Falco Talon
We can use helm to deploy Falco Talon from the official helm chart repository, part of falcosecurity organization:
```
git clone https://github.com/falcosecurity/charts.git
git checkout tags/falco-talon-0.3.0
cd charts/charts/falco-talon/
```

Remove the existing rule and download (wget) our customized Falco Talon [rules.yaml](https://raw.githubusercontent.com/IgorEulalio/falco-workshops/refs/heads/main/talon/talon-rules.yml) file containing the Talon rules in response to specific Falco events:
```
rm rules.yaml
wget -O rules.yaml https://raw.githubusercontent.com/IgorEulalio/falco-workshops/refs/heads/main/talon/talon-rules.yml
```

Now, we'll be deploying Falco Talon using the local chart, therefore Helm will use the rules file downloaded above as the official rules.
<br/><br/>
Install Falco-Talon:
```
helm upgrade --install falco-talon -n falco --create-namespace .
```
Falco Talon will be deployed with the custom rules content within the Helm-formatted rules.yaml file that is now present on your filesystem.
<br/><br/>
If the falco-talon pods are running, we can progress to the next lab scenario:
```
kubectl get pods -n falco -w | grep talon
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

## Test Process

Create an Ubuntu test workload
```
kubectl apply -f https://raw.githubusercontent.com/nigel-falco/oss-security-workshop/main/k03-rbac/ubuntu-pod.yaml
```

```
kubectl exec -it $(kubectl get pods -l app=ubuntu -o jsonpath='{.items[0].metadata.name}') -- /bin/bash
```

```
kubectl logs -n falco -l app.kubernetes.io/instance=falco-talon --max-log-requests=10
```

```
kubectl get events -n default -w
```

```
kubectl logs -l app.kubernetes.io/name=falco -n falco -c falco -f | grep -E "xmrig|mitre"
```
