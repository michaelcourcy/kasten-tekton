# Kasten Tekton

![Kasten Tekton Logo](doc/media/kasten-tekton.png)

Kasten-Tekton demonstrates how you can encapsulate Kasten Action and Policy in Tekton Task. 

With this approach you can orchestrate complex migration scenario with Tekton or include Kasten Action into your
existing pipleline.

## Disclaimer 

This project is just given as example and is not intended for support, even if you bought Kasten licences.

## The Kasten Tekton Task 

Today we implement 4 Kasten Tekton task 

- BackupAction 
- ExportAction 
- ImportAction
- RestoreAction 

They all have params and result that can consume each other in a pipeline (see the pipeline folder)

## Test it 

We suppose you already have 2 clusters that we'll call source and destination and Kasten install on both of them. 

You're going to install tekton on one of these cluster or spin up a new cluster. We'll demonstrate the latter.

### Spin up kind cluster with tekton 

```
git clone git@github.com:tektoncd/plumbing.git
cd plumbing/hack 
# -k if for docker without it's podman
./tekton_in_kind.sh -k 
# the script launch port-forward on 9097 hence you can directly
open http://localhost:9097
# otherwise you can execute the port-forward 
# kubectl port-forward service/tekton-dashboard -n tekton-pipelines 9097:9097 
```

### install the Kasten Tekton Tasks and the migration pipeline

```
kubectl create -f tasks/
kubectl create -f pipelines/
```

### Test a migration

Create the kubeconfig workspaces for source and destination 

```
kubectl config use-context <source>
./sa-config.sh aro-cluster https://<source-api> kasten-io k10-k10 > source-config
kubectl config use-context <destination>
./sa-config.sh eks-cluster https://<destination-api> kasten-io k10-k10 > eks-config
kubectl config use-context kind-tekton
kubectl create secret generic source-config --from-file=kubeconfig=source-config
kubectl create secret generic destination-config --from-file=kubeconfig=destination-config
```

Create an app on destination
```
kubectl config use-context <destination>
helm repo add pacman https://shuguet.github.io/pacman/
helm repo update
helm install pacman pacman/pacman -n mcourcy-pacman --create-namespace --set service.type=LoadBalancer 
```

check you can access the pacman board and record a game in the high score.


Launch the migration of pacman from source to destination, edit the migration-pipeline-run.yaml file 
and change the targetstorageclass-name value.

```
kubectl create -f examples/pipeline-run-examples/migration-pipeline-run.yml
```

Check pacman app on the destination, control high score match with the source.