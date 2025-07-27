---
title: "Terraform, Infrastructure as Code management tool"
date: 2025-07-26T22:38:40+08:00
draft: true
tags: ["Continuous Integration", "Continuous Deployment", "DevOps", "SRE", "IaC"]
---

# IaC Concept
* fast deployment for business requirement
* infrastructure consistency
* security compliance automation

## Azure cloud infra architecture overview
<img src="/azure/microservices-architecture.svg" width="800">

https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/containers/aks-microservices/aks-microservices

## Proposed GitOps flow
* change request
* stakeholder notification, plan
* IaC modularization for reusable purpose
* version control, branch: `main`, `dev`, `new_net`, `new_db`, `stg`, `prd`, ...
* automating testing
* continuous deployment
* update process to related notificatition channel

### HCL Code example
```hcl
variable "aks_node_pool" {
  description = ""
  type = map(object({
    vm_size         = string
    node_count      = number
    zones           = list(string)
    os_disk_size_gb = number
    os_disk_type    = string
  }))
  default = {
    "mqpool" : { vm_size = "Standard_DS4_v2", node_count = 6, zones = ["1", "2"], os_disk_size_gb = 64, os_disk_type = "Ephemeral" }
    "tbcorepool" : { vm_size = "Standard_DS3_v2", node_count = 3, zones = ["1", "2"], os_disk_size_gb = 64, os_disk_type = "Ephemeral" }
    "tbrepool" : { vm_size = "Standard_DS3_v2", node_count = 25, zones = ["1", "2"], os_disk_size_gb = 64, os_disk_type = "Ephemeral" }
    "tbtranspool" : { vm_size = "Standard_DS3_v2", node_count = 12, zones = ["1", "2"], os_disk_size_gb = 64, os_disk_type = "Ephemeral" }
    "tbjspool" : { vm_size = "Standard_DS2_v2", node_count = 6, zones = ["1", "2"], os_disk_size_gb = 64, os_disk_type = "Ephemeral" }
  }
}
module "aks" {
  source                               = "../../modules/azure/kubernetes"
  resource_group                       = var.resource_group
  location                             = var.location
  vnet_app_name                        = local.vnet_app_name
  subnet_aks_name                      = var.subnet_aks_name
  aks_name                             = local.aks_name
  aks_namespace_admin_group_object_ids = var.aks_namespace_admin_group_object_ids
  aks_admin_group_object_ids           = var.aks_admin_group_object_ids
  apg_id                               = module.apg.apg_id
  acr_id                               = module.acr.acr_id
  aks_tags                             = var.aks_tags
  aks_default_node_pool                = var.aks_default_node_pool
  aks_node_pool                        = var.aks_node_pool

  vnet_depends_on = [
    module.network.vnet_app_name,
    module.apg.apg_id,
    module.acr.acr_id
  ]
}
```

* for more detail, please check: https://github.com/chienfuchen32/terraform-azure

### Further IaC with documentation tool

<img src="/terraform/terraformproviderdevelopment-250116151500-84e30bd1.pdf" width="800">

* for more detail, please check: [Cloud Native Taiwan User Group meetup #65 slide](/terraform/terraformproviderdevelopment-250116151500-84e30bd1.pdf)

## Ref
* https://www.redhat.com/en/topics/automation/what-is-infrastructure-as-code-iac
* https://about.gitlab.com/topics/gitops/
* https://developer.hashicorp.com/terraform/language/modules
* https://developer.hashicorp.com/terraform/language/tests
* https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/teams?view=azure-devops