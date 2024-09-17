# ContainerDays 2024 Talk «Evolving GitOps: Harnessing Kubernetes Resource Model for 5G Core»: Config Hydration Examples

## Authors

Please feel free to approach us with feedback and questions!

Alexander North <alexander.north1@swisscom.com>
Ashan Senevirathne <ashan.senevirathne@swisscom.com>
Joel Studler <joel.studler@swisscom.com>

Contact us on slack:

- <https://cloud-native.slack.com>
- <https://kubernetes.slack.com>
- <https://nephio.slack.com>
- <https://netdev-community.slack.com>

## Prepare Demo Environment

### Kind with CRDs and local NetBox Operator instance

```bash
kind create cluster
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
pushd operator
make install
popd

pushd /tmp
git clone https://github.com/netbox-community/netbox-operator || echo "already exists"
pushd netbox-operator
make install
make create-kind
make deploy-kind
popd
popd
```

### Create the Prefix

This NetBox Prefix is required as an entrypoint for all our automation.

```bash
cat <<EOF | kubectl apply -f -
---
apiVersion: netbox.dev/v1
kind: Prefix
metadata:
  name: parent-prefix
spec:
  prefix: "10.0.0.0/16"
  preserveInNetbox: true
EOF
kubectl wait prefix parent-prefix --for=condition=Ready
kubectl get prefix parent-prefix
kubectl delete prefix parent-prefix  # to reduce confusion during the demo
```

## Hydration

Here is where you will run the hydration using Ansible Playbook, Kform or the Kubernetes Operator. See the README.md in the subfolder.

You can manually test the functionality of MetalLB (without hydration) using:

```bash
kubectl apply -f expected-io/output-ipaddresspool.yaml
```

## Watch relevant resources in terminal

```bash
watch -n .2 'echo "ConfigMaps:\n" && kubectl get cm -A -l demo=input && echo "\n\nConfigurations:\n" && kubectl get configurations -A && echo "\n\nNetBox PrefixClaims, NetBox Prefixes, MetalLB IPAddressPools:\n" && kubectl get prefixclaim,prefix,ipaddresspool -A'
```

## Test after hydration is performed

```bash
for i in {1..16}; do
  kubectl create svc loadbalancer kform-test-${i} --tcp 80:80
done
kubectl get svc | grep -v kubernetes
```

## Cleanup

### Cleanup all test svc

```bash
kubectl get svc -oname | grep test | xargs -n1 kubectl delete
```

### Cleanup everything

```bash
killall main
kind delete cluster
```
