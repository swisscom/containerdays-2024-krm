# Config hydration using Kform

## About Kform

<https://github.com/kform-dev>

## Prepare kform

```bash
pushd /tmp
git clone https://github.com/kform-dev/kform || echo "already exists"
pushd kform
make all
popd
git clone https://github.com/kform-providers/kubernetes || echo "already exists"
pushd kubernetes
make all
popd
popd
export PATH="/tmp/kform/bin:$PATH"
export KFORM_PROVIDER_KUBERNETES=/tmp/kubernetes/bin/kform-provider-kubernetes
```

## Hydrate the configuration and apply to cluster

Note: In a production scenario, this should be executed using some kind of workflow engine such as Argo Workflows <https://argoproj.github.io/workflows/> and triggered via Argo Events on changes of the high level resource <https://argoproj.github.io/argo-events/sensors/triggers/k8s-object-trigger/>.

```bash
pushd kform
kform init .
cat input-config.yaml
kform apply .
```

### Test after hydration is performed

```bash
for i in {1..16}; do
  kubectl create svc loadbalancer kform-test-${i} --tcp 80:80
done
kubectl get svc | grep -v kubernetes
for i in {1..16}; do
  kubectl delete svc kform-test-${i} > /dev/null
done
```

### Clean up kform

Kform handles state in the .kform/kform-inventory.yaml as well as in a ConfigMap in the kform-system namespace on the kube-api.

```bash
kform destroy .
rm .kform/kform-inventory.yaml
popd
```
