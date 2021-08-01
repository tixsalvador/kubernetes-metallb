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

Configure metallb

Create configmap yaml [metallb_l2.yml]

```sh
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.10.10.30-10.10.10.40
```

Deploy [metallb_l2.yml]

```sh
$  kubectl apply -f  https://raw.githubusercontent.com/tixsalvador/kubernetes-metallb/main/metallb_l2.yml
```

Create the nginx pod deployment [nginx_plain.yaml]

```sh
$  kubectl create -f https://raw.githubusercontent.com/tixsalvador/kubernetes-metallb/main/nginx_plain.yaml
```

Check status

```sh
$  kubectl  get svc nginx
NAME    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx   LoadBalancer   10.110.227.145   10.10.10.30   80:31875/TCP   33m
```

Notice the EXTERNAL-IP has an IP address 10.10.10.30 which we set on [metallb_l2.yml]

Add iptables rule for forwarding port 80 to the internal load balancer

```sh
$  iptables -A PREROUTING -t nat -i eth2 -p tcp --dport 80 -j DNAT --to 10.10.10.30:80
$  iptables -A FORWARD -p tcp  -i eth2  --dport 80 -j ACCEPT
$  iptables -A POSTROUTING -t nat -p tcp -d 10.10.10.30 --dport 80 -j SNAT --to-source 10.10.10.10
```

Where: 10.10.10.30 = LoadBalancer IP / 10.10.10.10 eth1 of my internal network

[metallb_l2.yml]: https://github.com/tixsalvador/kubernetes-metallb/blob/main/metallb_l2.yml
[nginx_plain.yaml]: https://github.com/tixsalvador/kubernetes-metallb/blob/main/nginx_plain.yaml

## SOURCE
https://metallb.universe.tf/installation/
