# Config hydration using Ansible + Jinja2

## About Ansible and Jinja2

<https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html>

## Create high level intent

Note: In a production scenario, this should be applied to the cluster using GitOps via tools like Argo CD or Flux CD.

```bash
pushd ansible
cat input/input-config.yaml
kubectl apply -f input/input-config.yaml
```

## Hydrate the configuration and apply to cluster

Note: In a production scenario, this should be executed using some kind of workflow engine such as Argo Workflows <https://argoproj.github.io/workflows/> and triggered via Argo Events on changes of the high level resource <https://argoproj.github.io/argo-events/sensors/triggers/k8s-object-trigger/>.

```bash
ansible-playbook playbook.yaml
```

### Test after hydration is performed

```bash
for i in {1..16}; do
  kubectl create svc loadbalancer ansible-test-${i} --tcp 80:80
done
kubectl get svc | grep -v kubernetes | grep -v postgres | grep -v netbox
for i in {1..16}; do
  kubectl delete svc ansible-test-${i} > /dev/null
done
```

### Clean up ansible

Since we're leveraging Kubernetes Garbage Collection using ownerReferences, we can just delete the high level intent to clean up. More info: <https://kubernetes.io/docs/concepts/architecture/garbage-collection/>

```bash
kubectl delete -f input/input-config.yaml
popd
```
