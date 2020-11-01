# Metallb kubernetes load balancer

#### Install metallb on kubernetes cluser

Prepartion:  
Since I'm using kube-proxy. I need to enable strict arp.

```sh
$  kubectl edit configmap -n kube-system kube-proxy
```

set strictARP: true

```sh
ipvs:
      excludeCIDRs: null
      minSyncPeriod: 0s
      scheduler: ""
      strictARP: true
      syncPeriod: 0s
      tcpFinTimeout: 0s
      tcpTimeout: 0s
```

or to automate this.

```sh
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

Apply the manifest

```sh
$  kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/namespace.yaml
$  kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/metallb.yaml
$  kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```
