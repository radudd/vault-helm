## About

This repo is a fork from the official HashiCorp Vault Helm charts and is optimised for OpenShift. It is different in the following aspects:

* OpenShift Routes are of type Re-encrypt and not Passthrough
* Services are signed using OpenShift CA Signer
* There is the possibility to use [Vault Bootstrap](github.com/radudd/vault-bootstrap). This is an Golang batch script that can peform the following actions:
** Initialize Vault
** Save Vault root token and unseal keys - for the moment this is limited to save into K8S secret, which is it insecure. However, the script should be extended to use other external secure stores, like AWS KMS, Azure KeyVault, etc.
** Unsealing Vault 
** Enable Vault K8S authentication

## HA Scenario

The HA scenario is optimised for using Consul as a backend, with Vault connecting directly to Consul servers and through agents. This is due to the fact that in order to deploy Consul agents on OpenShift an additional SCC to allow pods to run Host Network would be required.

*DISCLAIMER*: HashiCorp recommends using in general a Consul Agent when Consul is used as backend storage for Vault.

In an Vault deployment using SSL, the usage of RAFT storage has some limitation because the imposibility of the OpenShift CA signer to generate SSL certificates with SANs. When using RAFT storage, the SSL certificates must match both the Vault instance and Vault Load Balancer. 

## Deploy

If deploying HA Vault with Consul backend, first update the `override-ha.yaml` according to your needs.

Then fetch Consul Helm chart dependency.

```
helm dep update
```

Finally install Vault 

```
helm install vault . -f override-ha.yaml
```
