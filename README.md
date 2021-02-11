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

