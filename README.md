# Ray

## K8s Prereqs

* K8s cluster


* Helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
source: https://helm.sh/docs/intro/install/


## Ray Prereqs

* Deploy KubeRay CRDs:
```bash
helm repo add kuberay https://ray-project.github.io/kuberay-helm/
helm repo update

# Install both CRDs and KubeRay operator v1.0.0-rc.0.
helm install kuberay-operator kuberay/kuberay-operator --version 1.0.0-rc.0

# Confirm that the operator is running in the namespace `default`.
kubectl get pods
```
```bash
kubectl get crd
```
source: https://docs.ray.io/en/latest/cluster/kubernetes/getting-started/raycluster-quick-start.html


* python SDK
```bash
pip3 install --upgrade pip
pip3 install "ray[default]"
pip3 install "ray[data,train,tune,serve]"
# pip3 install "ray[rllib]" # todo
```
source: https://docs.ray.io/en/latest/ray-overview/installation.html


## RayCluster:
2 job submission methods (via head node pod)

inspect yaml
```bash
vi kube/static_cluster.yaml
```

create static cluster
```bash
kubectl apply -f kube/static_cluster.yaml
kubectl get rayclusters
```

### Method 1: Execute a Ray job in the head Pod

```bash
export HEAD_POD=$(kubectl get pods --selector=ray.io/node-type=head -o custom-columns=POD:metadata.name --no-headers)
echo $HEAD_POD

# Print the cluster resources.
kubectl exec -it $HEAD_POD -- python -c "import ray; ray.init(); print(ray.cluster_resources())"
```

### Method 2: Submit a Ray job to the RayCluster via ray job submission SDK

```bash
kubectl get service raycluster-xgboost-benchmark-head-svc
```
```bash
# Run the following blocking command in a separate shell.
kubectl port-forward --address 0.0.0.0 service/raycluster-xgboost-benchmark-head-svc 8265:8265
```
on second terminal
```bash
ray job submit --address http://localhost:8265 -- python -c "import ray; ray.init(); print(ray.cluster_resources())"
```
other example
```bash
ray job submit --address=http://localhost:8265 -- echo hello world
```
cleanup
```bash
kubectl delete -f kube/static_cluster.yaml
```

## RayJob
A Kubernetes Job runs ray job submit to submit a Ray job to the RayCluster.


inspect yaml
notice how configmap is used as a shorthand for embedding code into a new container
```bash
vi kube/ray_job.yaml
```
You could optionally pull from official images, containereize logic and push to docker image and execute (without configmap)


inspect Dockerfile
```
https://github.com/ray-project/ray/tree/master/docker
https://github.com/ray-project/ray/blob/master/docker/ray/Dockerfile
```

submit rayjob
```bash
kubectl apply -f kube/ray_job.yaml
kubectl logs -l=job-name=rayjob-sample
```

inspect resource execution
```bash
watch kubectl get po
kubectl get raycluster
kubectl get rayjobs
kubectl get rayjobs.ray.io rayjob-sample -o json | jq '.status.jobStatus'
```
Simulate containereized workload


## RayService
```
kubectl get rayservices
```



## future examples
https://docs.ray.io/en/latest/cluster/kubernetes/examples/ml-example.html#kuberay-ml-example
https://docs.ray.io/en/latest/ray-core/examples/highly_parallel.html
https://docs.ray.io/en/latest/ray-core/examples/map_reduce.html
