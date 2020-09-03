# vault-kubernetes-poc

Kubernetes Consumer Application for Vault. Deploys a Pod with a Vault Agent Container and a HelloGo container. The Vault Agent is in charge of connecting to the Vault cluster and refresh dynamic credentials via Consul Template to retrieve secrets.