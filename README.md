# Kubernetes.Installation

## Prerequisites I've used Ubuntu locally and for remote server. So all commands for Ubuntu.

### Check ssh connection to your remote server:

```bash
ssh your_user@remote_ip
```
___

### Install golang https://go.dev/doc/install

  ```bash
   wget https://go.dev/dl/go1.20.1.linux-amd64.tar.gz
   rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.4.linux-amd64.tar.gz
   ```

   To use specific path add in the end:

  ```bash
   vim ~/.profile 
   export PATH=$PATH:/usr/local/go/bin
   source ~/.profile
   go version
   ```
___

### Install kubectl

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version
```
___

### Install k9s to maintain your cluster(it's like Total commander for cluster) https://k9scli.io/

Go to https://github.com/derailed/k9s/releases and choose nessesary architecture For example:

```bash
wget https://github.com/derailed/k9s/releases/download/v0.26.7/k9s_Linux_x86_64.tar.gz
tar -C /usr/local/bin -xzf k9s_Linux_x86_64.tar.gz
k9s
```
___

### Install k3s (lightweight kubernetes) on remote host https://k3s.io/

#### Requirements:

https://docs.k3s.io/installation/requirements
```bash
curl -fL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644 --disable traefik --disable servicelb 
```
NOTE! We disable default ingress-controller and we weill install nginx ingess controller. 
Also disabling loadbalancer if you want to play with a load-balancer you can use https://metallb.universe.tf/

To check cluster:
```bash
kubectl get pods -A
```

check config (file to get access to cluster)
```bash
cat /etc/rancher/k3s/k3s.yaml 
```
Then locally (copy this file to your local machine). For example:
```bash
mkdir ~/.kube/ scp -o ProxyCommand="ssh -W %h:%p -q jump_sa@178.124.206.53" root@192.168.203.XX:
/etc/rancher/k3s/k3s.yaml ~/.kube/config
```
When you start k9s you should see this config

Then we should get remote 6443 to our local port.
```bash
ssh -o ProxyCommand="ssh -W %h:%p -q jump_sa@178.124.206.53" -L 6443:127.0.0.1:6443 root@192.168.203.XX -f -N
```

Check cluster locally:
```bash
kubectl get pods -A
```
___

### Deploy ingress controller 

browser => hosts => 178.124.206.53 (nginx) -> ingress (app.k8s-18.sa) ->service -> pod (container)

You can use files nginx.yaml and nginx-controller.yaml in repo. Or
```bash
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml -O
nginx-controller.yaml
kubectl get pods -A
```
Then add NodePort to service (exposes the service on a static port on the node's IP address)
