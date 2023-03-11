# Kubernetes

This is the result of me fiddling with Kubernetes, ArgoCD, Helm Tekton...

## Setup Minikube

_Will make Kubernetes available on localhost._

There is an excelent guide here:
https://minikube.sigs.k8s.io/docs/start/

## Setup Tekton

_Will enable pipelines in Kubernetes._

There is an excelent guide here:

https://tekton.dev/docs/installation/pipelines/

Install latest Tekton:

```sh
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

And watch until installation is complete:

```sh
kubectl get pods --namespace tekton-pipelines --watch
```

## Setup ArgoCD

_Will detect changes in Git and apply those changes to Kubernetes._

There is an excelent guide here:

https://argo-cd.readthedocs.io/en/stable/getting_started/

This is section is basically a slimmer version of that guide.

**Create a namespace and install ArgoCD in it:**

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Install ArgoCD CLI:**

https://argo-cd.readthedocs.io/en/stable/cli_installation/

I use install with `curl`:

```sh
curl -SL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

**Enable access to ArgoCD:**

```sh
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

I change the `password` to `thepassword`. First by getting the encryped value:

```sh
argocd account bcrypt --password thepassword
# -> $2a$10$alzOPpjYB88566WHDsDac.b1EE4XcOrzB.qAVwp7liF4kvW2TVo2S
```

And applying the encryped value:

```sh
kubectl -n argocd patch secret argocd-secret \
  -p '{"stringData": {
    "admin.password": "$2a$10$alzOPpjYB88566WHDsDac.b1EE4XcOrzB.qAVwp7liF4kvW2TVo2S",
    "admin.passwordMtime": "'$(date +%FT%T%Z)'"
  }}'
```

Login to ArgoCD:

```sh
argocd \
 --insecure login \
 localhost:8080 \
 --username admin \
 --password thepassword
```

You can also login via GUI by browsing to: https://localhost:8080/

**Let ArgoCD apply Tekton**

Let ArgoCD monitor this repository and apply it to this cluster:

```sh
kubectl config set-context --current --namespace=argocd

argocd app create tekton-tasks --repo https://github.com/tomasbjerre/kubernetes.git --path tekton --dest-server https://kubernetes.default.svc --dest-namespace default
```

# Commands

| **Command**          | **Explanation**                                        |
| -------------------- | ------------------------------------------------------ |
| `minikube start`     | Start Minikube                                         |
| `minikube dashboard` | Start Minikube dashboard so that you can browse to it. |
