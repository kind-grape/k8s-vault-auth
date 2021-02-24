# vault-kubernetes-poc

### Connectivity

- the Kubernetes API can connect to the Vault Agent injector service on port `443`, and
  the injector can connect to the Kubernetes API,
- Vault can connect to the Kubernetes API,
- Pods in the Kubernetes cluster can connect to Vault.

### Kubernetes and Vault Configuration

- Kubernetes auth method should be configured and enabled in Vault,
- Pod should have a service account,
- desired secrets exist within Vault,
- the service account should be bound to a Vault role with a policy enabling access to desired secrets.

### how does it work??
The Vault Agent Injector alters pod specifications to include Vault Agent
containers that render Vault secrets to a shared memory volume using
[Vault Agent Templates]

The injector is a [Kubernetes Mutation Webhook Controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/).
The controller intercepts pod events and applies mutations to the pod if annotations exist within
the request.

In the example use case, it creates a deployment that mounts a Kubernetes ConfigMap containing Vault Agent configuration files. For a complete list of the Vault Agent configuration settings

- `vault.hashicorp.com/agent-inject` - configures whether injection is explicitly
  enabled or disabled for a pod. This should be set to a `true` or `false` value.
  Defaults to `false`.

- `vault.hashicorp.com/agent-configmap` - name of the configuration map where Vault
  Agent configuration file and templates can be found.

The deployment will also create a Vault agent container within the same pod of the deployment. Vault agent will take care of the auto-auth of the k8s deployment to the Vault cluster. In this example, the vault agent would also fetch the secret engine every 5min (https://www.vaultproject.io/docs/agent/template#non-renewable-secrets) so that when the secret updated in Vault, it will also reflect the change in the container run time.

### Configuring Vault to enable K8s Auth
- Prerequisite: In order for Vault cluster to authenticate with the K8s cluster, the K8s cluster would need to have a service account/service account token setup first. Please contact the cluster admin to have that set up.
The following 3 items would be needed when configuring the k8s auth method
token_reviewer_jwt - which can be obtained from the json field of the service account
kubernetes_ca_cert - which can be obtained from the json field of the service account

Following commands can be used to get those fields of the service account attributes
```text
kubectl get secret svc-tke-tevdev-token-4w6qw -o jsonpath="{.data.token}" | base64 --decode; echo
kubectl get secret svc-tke-tevdev-token-4w6qw -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo
```

```text
$ vault write auth/kubernetes/config \
    token_reviewer_jwt="<your reviewer service account JWT>" \
    kubernetes_host=https://<k8s_host>:<your TCP port or blank for 443> \
    kubernetes_ca_cert=@ca.crt
```

After configuring the auth method, Vault team would then need to create a role within Vault, which would then grant the matching service account in k8s to be given a policy to consume secret in Vault

```text
vault write auth/kubernetes/role/demo \
    bound_service_account_names=svc-tke-tevdev \
    bound_service_account_namespaces=tev-dev \
    policies=appa-kv-ro \
    ttl=1h
```


