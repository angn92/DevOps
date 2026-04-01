# Kubectl commands

1. Basic commands for getting information

```bash
kubectl cluster-info # view cluster information

kubectl get nodes # list all nodes in cluster

kubectl get pods # list pods in current namespace

kubectl get services # or kubectl get svc

kubectl get deployments # list deployments

kubectl get all # get most common resources (pods, svc, deploy, etc.)
```

2. Namespaces

```bash
kubectl get namespaces # or kubectl get ns

kubectl create namespace <namespace_name> # create namespace

kubectl delete namespace <namespace_name> # delete namespace

kubectl config set-context NAME [--cluster=cluster_name] [--user=user_name] [--namespace=namespace] 
# define cluster/user/namespace for context

# Create new context:
kubectl config set-context <context_name> \
    --cluster=my-cluster \
    --user=my-user \
    --namespace=default 

# Update an existing context
kubectl config set-context my-context --namespace=dev

# Switch to created context
kubectl config use-context my-context

# View contexts
kubectl config get-contexts
```
3. Pods
```bash
kubectl get pods -o wide # show more details (IP, node, etc.)

kubectl describe pod <pod_name> # detailed info about pod

kubectl logs <pod_name> # view logs

kubectl logs -f <pod_name> # stream logs

kubectl exec -it <pod_name> -- /bin/sh # open shell inside pod

kubectl delete pod <pod_name> # delete pod

kubectl run <pod_name> --image=nginx # quickly create a pod
```

4. Deployment
```bash
kubectl create deployment <deployment-name> --image=nginx 
# create deployment

kubectl get deployments # list deployments

kubectl describe deployment <deployment-name> # detailed info

kubectl scale deployment <deployment-name> --replicas=3 
# scale number of pods

kubectl set image deployment/<deployment-name> nginx=nginx:latest 
# update container image

kubectl rollout status deployment/<deployment-name> 
# check rollout status

kubectl rollout history deployment/<deployment-name> 
# see rollout history

kubectl rollout undo deployment/<deployment-name> 
# rollback to previous version

kubectl delete deployment <deployment-name> 
# delete deployment
```

5. Services (svc)
```bash
kubectl get svc # list services

kubectl expose deployment <deployment-name> --type=ClusterIP --port=80 
# expose deployment internally

kubectl expose deployment <deployment-name> --type=NodePort --port=80 
# expose externally via node port

kubectl expose deployment <deployment-name> --type=LoadBalancer --port=80 
# expose using cloud load balancer

kubectl describe svc <service-name> # detailed info

kubectl delete svc <service-name> # delete service
```

6. ConfigMap
```bash
kubectl create configmap <config-name> --from-literal=key=value 
# create from key-value

kubectl create configmap <config-name> --from-file=config.txt 
# create from file

kubectl get configmaps # list configmaps

kubectl describe configmap <config-name> # detailed info

kubectl delete configmap <config-name> # delete configmap
```

7. Secrets
```bash
kubectl create secret generic <secret-name> --from-literal=password=1234 
# create secret

kubectl get secrets # list secrets

kubectl describe secret <secret-name> # view metadata (not values)

kubectl delete secret <secret-name> # delete secret
```

8. Ingress (Route)
```bash
kubectl get ingress # list ingress resources

kubectl describe ingress <ingress-name> # detailed info

kubectl delete ingress <ingress-name> # delete ingress

kubectl apply -f ingress.yaml # create ingress from file
```

9. For YAML files
```bash
kubectl apply -f file.yaml # create/update resources

kubectl delete -f file.yaml # delete resources

kubectl create -f file.yaml # create (won’t update existing)

kubectl replace -f file.yaml # replace existing resource

kubectl diff -f file.yaml # show differences before applying
```

10. Debugging & troubleshooting
```bash
kubectl describe pod <pod_name> # check events/errors

kubectl logs <pod_name> # check logs

kubectl get events # cluster events

kubectl top pods # resource usage (CPU/memory)

kubectl top nodes # node usage
```

11. Port forwarding
```bash
kubectl port-forward pod/<pod_name> 8080:80 
# access pod locally

kubectl port-forward svc/<service-name> 8080:80 
# access service locally
```