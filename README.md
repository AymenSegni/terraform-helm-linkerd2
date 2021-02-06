# deploy-linkerd-terraform-helm

Deploy Linkerd2 using Terraform Helm Provider. 
Linkerd: Ultra light, ultra simple and ultra powerfu service mesh
Linkerd adds security, observability, and reliability to Kubernetes, without the complexity. CNCF-hosted and 100% open source.

## Terraform Linkerd Module

This module handles Linkerd creation and configuration HA mode and Trusted Anhcor Certificate.
The resources creation that this module will create/trigger are:
- Create a Linkerd control plan with the provided addons
- Setting High-Availability on demande for production cluster using a file values-ha.yaml that overrides some default values as to set things up under a high-availability scenario, analogous to the `--ha` option in linkerd install. Values such as higher number of replicas, higher memory/cpu limits and affinities are specified in that file.
- Create trusted Anchor identity  certificate using the ECDSA P-256 algorithm

## Compatibility

This module is meant for use with Terraform 0.12. If you haven't
[upgraded][terraform-0.12-upgrade] and need a Terraform
0.11.x-compatible version of this module, the last released version
intended for Terraform 0.11.x is [3.0.0].

## Usage

There are multiple usage examples but simple usage is as follows:

```hcl

# kubernetes and Helm provider must be explicitly specified like the following.

// aks cluster

data "azurerm_kubernetes_cluster" "dev_aks_cluster" {
  name                = "dev"
  resource_group_name = "aks_dev_resource_group"
}

// Helm provider 

provider "helm" {
  kubernetes {
    host                   = data.azurerm_kubernetes_cluster.dev_aks_cluster.kube_admin_config.0.host
    client_certificate     = base64decode(data.azurerm_kubernetes_cluster.dev_aks_cluster.kube_admin_config.0.client_certificate)
    client_key             = base64decode(data.azurerm_kubernetes_cluster.dev_aks_cluster.kube_admin_config.0.client_key)
    cluster_ca_certificate = base64decode(data.azurerm_kubernetes_cluster.dev_aks_cluster.kube_admin_config.0.cluster_ca_certificate)
  }
  alias = "aks-dev"
}

// Deploy Linkerd on DEV cluster with disabling HA Mode

module "dev_linkerd" {
  source            = "AymenSegni/deploy-linkerd-terraform-hel"
  enable_linkerd_ha = false 
  providers = {
    helm = helm.aks-dev
  }
}
```
Then perform the following commands on the root folder:

- `terraform init` to get the plugins
- `terraform plan` to see the infrastructure plan
- `terraform apply` to apply the infrastructure build
- `terraform destroy` to destroy the built infrastructure
