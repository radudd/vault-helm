## About

This repo is a fork from the official HashiCorp Vault Helm charts and is optimised for OpenShift. It is different in the following aspects:

* OpenShift Routes are of type Re-encrypt and not Passthrough
* Services are signed using OpenShift CA Signer
* There is the possibility to use [Vault Bootstrap](github.com/radudd/vault-bootstrap). This is an Golang batch script that can peform the following actions:

1. Initialize Vault
2. Save Vault root token and unseal keys - for the moment this is limited to save into K8S secret, which is it insecure. However, the script should be extended to use other external secure stores, like AWS KMS, Azure KeyVault, etc.
3. Unsealing Vault 
4. Enable Vault K8S authentication

## HA Scenario

You can find an example on using Consul as a storage backend for HA. In this example, Vault connects directly to Consul servers and *NOT* through agents. This is due to the fact that in order to deploy Consul agents on OpenShift an additional SCC to allow pods to run Host Network would be required.

> :warning: **DISCLAIMER**: HashiCorp recommends using in general a Consul Agent when Consul is used as backend storage for Vault. Details [here](https://learn.hashicorp.com/tutorials/vault/ha-with-consul#consul-client-agent-configuration)

> :warning: **DISCLAIMER**: Consul doesn't use ACLs for our first scenario, so Consul endpoint must be secured. Anyone with anonymous access to Consul might *delete* Vault data. For production usage, you should implement ACLs. Use the following references: [Vault](https://www.vaultproject.io/docs/configuration/storage/consul#acls) and [Consul](https://www.consul.io/docs/k8s/helm#v-global-acls-bootstraptoken). You can also check the [minimal implementation](#Consul-HA-ACL) from this repository.

In an Vault deployment using SSL, the usage of RAFT storage has some limitation because the imposibility of the OpenShift CA signer to generate SSL certificates with SANs. When using RAFT storage, the SSL certificates must match both the Vault instance and Vault Load Balancer. 

More example for different storage backends will follow ...

## Deploy

### Consul HA No-ACL

If deploying HA Vault with Consul backend(no agents and no ACLs), first update the `override/vault-ha-consul-noacl.yaml` according to your needs.

Then fetch Consul Helm chart dependency.

```
helm dep update
```

Finally install Vault 

```
helm install vault . -f override/ha-consul-noacl.yaml
```

### Consul HA ACL

First deploy Consul 

```
helm repo add hashicorp https://helm.releases.hashicorp.com
```

```
helm install consul hashicorp/consul -f override/consul-acl.yaml
```

Extract the generated ACL token

```
TOKEN=`oc get secret consul-consul-bootstrap-acl-token -o template --template '{{.data.token}}'|base64 -d`
```

Add this token to Vault override

```
sed -i "/token/s/dummytoken/$TOKEN" override/vault-ha-consul-acl.yaml
```

Finally install Vault
```
helm install vault . -f override/vault-ha-consul-acl.yaml
```
