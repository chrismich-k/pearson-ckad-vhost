# Because /etc/hosts won't help

> My journey learning Kubernetes via the Pearson CKAD preparation: 
> Moving from Minikube/Calico to a live environment using Flannel, Nginx-Ingress, and wildcard DNS on a fixed-IP server.

---

### 🕵️ Why this repo exists
Most courses rely on `minikube` and local hacks like editing `/etc/hosts`. I decided to try a simple setup to enjoy DNS propagation, Ingress controllers, and automated TLS. Added some crude installation notes below to remember. I edited some of the Demos to work with name-based virtual hosting. And I'd like to point out to my future self that the TLS certificates were actually generated and working fine out of the box.

### 🛠 The Stack
*   **Infrastructure:** Bare-metal kubeadm (v1.3x) on a fixed-IP VPS.
*   **Networking**: Flannel (L3 Overlay) instead of Calico. They said it's simpler - we'll see.
*   **Traffic:** `ingress-nginx` acting as the gateway.
*   **DNS:** Global Wildcard A-Record (`*.k8s.yourdomain.tld`) pointing to the node.
*   **Certificates:** `cert-manager` with Let's Encrypt (HTTP-01 works fine).

### 🧱Installation

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

use an IP in an internal network for your nodes (e.g. vLAN by your Provider),
add `--apiserver-cert-extra-sans=<WAN_IP>` if you need to allow it to listen also on WAN (then Firewall it!).
```bash
kubeadm init \
  --apiserver-advertise-address=<VLAN_IP> \
  --pod-network-cidr=10.10.0.0/16
```

Flannel
```bash
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

edit to have it listen on the `<VLAN_IP>` 
```yaml
      containers:
      - args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=<WAN_IP>
```

```bash
kubectl apply -f kube-flannel.yml
```

Allow worker nodes on the same node as the control plane
```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

Ingress on a static public IP - I have no other http(s) services on this IP so fine and simple
```bash
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml
```

edit your static `<WAN_IP>`
```yaml
   type: NodePort
   externalIPs:
   - <WAN_IP>
```

```bash
kubectl apply -f deploy.yaml
``` 

Cert-Manager (LetsEncrypt)
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml
```

create `issuer.yaml`, use actual email address
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your@email.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
---
```

```bash
kubectl apply -f issuer.yaml
```

---
*Note: This is a learning log. It reflects my progress and might contain "non-exam" configurations. For original course instructions, see [README-ORIGINAL.md](./README-ORIGINAL.md).*
