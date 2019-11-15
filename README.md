# Demo: Secrets

### 1. Set-Up

```
brew install kubeseal
```

Install `sealed-secrets` chart into `kube-system`, e.g.

```
argocd app create sealed-secrets --repo https://kubernetes-charts.storage.googleapis.com --helm-chart sealed-secrets --revision '1.5.0' --dest-server https://kubernetes.default.svc --dest-namespace kube-system 
argocd app sync sealed-secrets
```

### 2. Create And Seal Secret

Create the secret:

```
echo -n bar | kubectl create secret generic mysecret --dry-run --from-file=foo=/dev/stdin -o json > mysecret.json
```

Seal it:

```
kubeseal --cert https://argo-cd-kubecon.apps.argoproj.io/secret/cert.pem < mysecret.json > mysealedsecret.json
```

Delete the un-sealed secret:

```
rm mysecret.json
```

Commit the changes:

```
git add mysealedsecret.json
git commit -am 'Add secret'
git push
```

### 3. Create An App

```
argocd app create my-secrets --repo https://github.com/gitops-workshop/argo-cd-demos.git --path . --revision 'master' --dest-server https://kubernetes.default.svc --dest-namespace default
argocd app sync my-secrets
```

Open the UI to observe the created secret, or use CLI:

```
kubectl -n default get secrets -o yaml
```

Decode `YmFy` using http://bit.ly/cnvcode.

### Clean-Up

```
argocd app delete my-secrets
argocd app delete sealed-secrets
```
