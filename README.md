# K8TRE Flower AI

This contains [K8TRE](https://github.com/k8tre/k8tre) ArgoCD configurations for deploying [Flower AI](https://flower.ai/).

## superlink-netbirdproxy

Deploy a FlowerAI SuperLink fronted by a NetBird proxy (ambassador pattern)

### ArgoCD Application

The ArgoCD Application requires some configuration.
Edit [`argocd-application-superlink-netbirdproxy.template.yaml`](./argocd-application-superlink-netbirdproxy.template.yaml):
- Replace `https://netbird.example.org` and `netbird.example.org` with your Netbird management URL
  (the Netbird client will connect to this) and host (used for allow-listing the Netbird managment host in a network policy).
- Set `spec.destination.name` if you are deploying the application in a different cluster from where ArgoCD is running
- Set `spec.destination.namespace` to change the Kubernetes namespace

Deploy the Application:
```sh
kubectl -n argocd apply -f argocd-application-superlink-netbirdproxy.yaml
```

Create a secret `netbirdproxy-secrets` containing your Netbird `setup-key` (replace `flowerai` with your namespace):
```sh
kubectl -nflowerai create secret generic netbirdproxy-secrets --from-literal=setup-key=SETUP_KEY
```

## Verifying the superlink connection

```sh
kubectl -nflowerai run flower-debug -it --rm --image=flwr/superlink:latest --command /bin/bash
```

```sh
mkdir -p ~/.flwr

cat << EOF > ~/.flwr/config.toml
[superlink.test]
address = "netbirdproxy:9093"
insecure = true
EOF

flwr supernode list test
```

If you're connecting over Netbird change `address = "netbirdproxy:9093"` to whatever the netbird hostname is for the Netbird superlink proxy.

To view the Haproxy statistics:

```sh
kubectl -nflowerai port-forward deploy/netbirdproxy 8404
```

and open http://localhost:8404 in your browser

## Deploying without K8TRE and ArgoCD

You might be able to deploy this independently of K8TRE and ArgoCD, but this is not supported nor tested:

```sh
kubectl apply -k superlink-netbirdproxy/ [--dry-run=server]
```
