Linkerd canary showcase
=======================

Testing canary releases with service mesh https://linkerd.io/2/tasks/canary-release/


Setup aks
---------

TODO

Get aks credentials 
-------------------

```
az login
az account set --subscription HCI_DEV_PD3
az aks get-credentials --resource-group mario_vejlupek_test --name mario-vejlupek-test
export KUBECONFIG=/Users/mario/.kube/config
k get no
```

Setup linkerd
-------------

```
curl -sL https://run.linkerd.io/install | sh
linkerd check --pre
linkerd install | kubectl apply -f -
linkerd check
kubectl -n linkerd get deploy
linkerd dashboard
```

Check traffic
-------------

```
linkerd -n linkerd top deploy/linkerd-web
```


Install flagger
---------------

```
kubectl apply -k github.com/weaveworks/flagger/kustomize/linkerd
kubectl -n linkerd rollout status deploy/flagger --watch
```


Demo
----

```
kubectl create ns test && kubectl apply -f https://run.linkerd.io/flagger.yml
kubectl -n test rollout status deploy podinfo --watch
kubectl -n test port-forward svc/frontend 8080
open http://localhost:8080
kubectl apply -f canary.yml 
kubectl -n test set image deployment/podinfo podinfod=quay.io/stefanprodan/podinfo:1.7.0

# Watch rollout
watch linkerd -n test stat deploy --from deploy/load  

# Roll new version
kubectl -n test set image deployment/podinfo podinfod=quay.io/stefanprodan/podinfo:1.7.1

# Fail deployment https://docs.flagger.app/usage/progressive-delivery
kubectl -n test run tester --image=quay.io/stefanprodan/podinfo:1.2.1 -- ./podinfo --port=9898 
kubectl -n test exec -it tester-564d456687-z759q sh
> watch curl http://podinfo-canary:9898/status/500
```
