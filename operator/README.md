# Config hydration using Kubernetes Operator

## About Kubernetes Operators

<https://kubernetes.io/docs/concepts/extend-kubernetes/operator/>

## Run Operator

Note: In a production scenario, this Operator would actively run on the cluster.

```bash
pushd operator
go build -o bin/config-hydration-operator cmd/main.go
bin/config-hydration-operator &
```

## Create input resource

Note: In a production scenario, this should be applied to the cluster using GitOps via tools like Argo CD or Flux CD.

```bash
cat config/samples/hydration-demo_v1alpha1_configuration.yaml
kubectl apply -f config/samples/hydration-demo_v1alpha1_configuration.yaml
```

### Test after hydration is performed

```bash
for i in {1..16}; do
  kubectl create svc loadbalancer operator-test-${i} --tcp 80:80
done
kubectl get svc | grep -v kubernetes
for i in {1..16}; do
  kubectl delete svc operator-test-${i} > /dev/null
done
```

### Clean up operator resources

Since we're leveraging Kubernetes Garbage Collection using ownerReferences, we can just delete the high level intent to clean up. More info: <https://kubernetes.io/docs/concepts/architecture/garbage-collection/>

```bash
kubectl delete -f config/samples/hydration-demo_v1alpha1_configuration.yaml
killall config-hydration-operator
popd
```
