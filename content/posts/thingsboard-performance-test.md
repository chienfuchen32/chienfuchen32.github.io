---
title: "Thingsboard Performance Test"
date: 2023-10-05T21:18:19+08:00
draft: true
tags: ["Kubernetes","Performance Test","Distributed System","High Availability","IoT", "Thingsboard"]
---
## Thingsboard
* [Micro service Architecture](https://thingsboard.io/docs/reference/)
<img src="/tb-perf/tb-architecture.png" width="1024">
* version v3.5.1
## Cloud Infrastructure Environment
* Azure Kubernetes v1.26.3. Tier: Standard
* Load Balancer. sku: Standard
* Azure Application Gateway. Tier: WAF V2 (auto scale instance 1-3)
* Azure PostgreSQL Flexible Server. version: 14.8 sku: 1 * D2ds_v4, 2vCores, 8GiB RAM, 128GiB 500 IOPS Disk)
* Azure Cache for Redis. sku: 1 * Premium P1 (6GB Cache Size)
* Azure Managed Cassandra. version: 4 sku: 9 * D8s_V5 (8 vCPUs, 32GB RAM, 2 * P30 1024GiB, 5000 IOPS, 200 MB/sec Disk)
* Azure Virtual Machine Scale Sets. detail were shown as below

* Deployment example
[Terraform Azure IaC](https://github.com/chienfuchen32/terraform-azure)
```bash
$ kubectl get node --show-labels=true
NAME                                  STATUS   ROLES   AGE     VERSION   LABELS
aks-monitor-22669933-vmss000000       Ready    agent   62d     v1.26.3   agentpool=monitor,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=monitor,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=5004d990-29c1-11ee-bc87-02438f727b86,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=system,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-monitor-22669933-vmss000000,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-monitor-22669933-vmss000001       Ready    agent   19d     v1.26.3   agentpool=monitor,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=monitor,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=5004d990-29c1-11ee-bc87-02438f727b86,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=system,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-monitor-22669933-vmss000001,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-mqpool-26362843-vmss000000        Ready    agent   67d     v1.26.3   agentpool=mqpool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D8s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=mqpool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=de7c7621-25d7-11ee-8d68-4ef9bda78873,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-mqpool-26362843-vmss000000,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D8s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-mqpool-26362843-vmss000003        Ready    agent   27d     v1.26.3   agentpool=mqpool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D8s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=mqpool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=de7c7621-25d7-11ee-8d68-4ef9bda78873,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-mqpool-26362843-vmss000003,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D8s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-mqpool-26362843-vmss000004        Ready    agent   27d     v1.26.3   agentpool=mqpool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D8s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=mqpool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=de7c7621-25d7-11ee-8d68-4ef9bda78873,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-mqpool-26362843-vmss000004,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D8s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-mqpool-26362843-vmss000005        Ready    agent   23h     v1.26.3   agentpool=mqpool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D8s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=mqpool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=de7c7621-25d7-11ee-8d68-4ef9bda78873,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-mqpool-26362843-vmss000005,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D8s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-mqpool-26362843-vmss000006        Ready    agent   23h     v1.26.3   agentpool=mqpool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D8s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=mqpool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=de7c7621-25d7-11ee-8d68-4ef9bda78873,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-mqpool-26362843-vmss000006,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D8s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-mqpool-26362843-vmss000007        Ready    agent   23h     v1.26.3   agentpool=mqpool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D8s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=mqpool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=de7c7621-25d7-11ee-8d68-4ef9bda78873,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-mqpool-26362843-vmss000007,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D8s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbcorepool-16715149-vmss000000    Ready    agent   67d     v1.26.3   agentpool=tbcorepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbcorepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=c6d2dc03-25d7-11ee-8704-8aa4fbc64cb3,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbcorepool-16715149-vmss000000,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbcorepool-16715149-vmss000001    Ready    agent   67d     v1.26.3   agentpool=tbcorepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbcorepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=c6d2dc03-25d7-11ee-8704-8aa4fbc64cb3,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbcorepool-16715149-vmss000001,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbcorepool-16715149-vmss000002    Ready    agent   67d     v1.26.3   agentpool=tbcorepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbcorepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=c6d2dc03-25d7-11ee-8704-8aa4fbc64cb3,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbcorepool-16715149-vmss000002,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbjspool-13871965-vmss000000      Ready    agent   67d     v1.26.3   agentpool=tbjspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbjspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=ecd20b35-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbjspool-13871965-vmss000000,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D2s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbjspool-13871965-vmss000001      Ready    agent   67d     v1.26.3   agentpool=tbjspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbjspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=ecd20b35-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbjspool-13871965-vmss000001,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D2s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbjspool-13871965-vmss000002      Ready    agent   67d     v1.26.3   agentpool=tbjspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbjspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=ecd20b35-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbjspool-13871965-vmss000002,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D2s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbjspool-13871965-vmss000003      Ready    agent   67d     v1.26.3   agentpool=tbjspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbjspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=ecd20b35-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbjspool-13871965-vmss000003,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D2s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbjspool-13871965-vmss000004      Ready    agent   67d     v1.26.3   agentpool=tbjspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbjspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=ecd20b35-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbjspool-13871965-vmss000004,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D2s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbjspool-13871965-vmss000005      Ready    agent   67d     v1.26.3   agentpool=tbjspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbjspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=ecd20b35-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbjspool-13871965-vmss000005,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D2s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss000000      Ready    agent   67d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss000000,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss000001      Ready    agent   67d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss000001,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss000002      Ready    agent   67d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss000002,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss000009      Ready    agent   26d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss000009,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000a      Ready    agent   26d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000a,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000b      Ready    agent   26d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000b,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000c      Ready    agent   26d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000c,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000d      Ready    agent   26d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000d,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000e      Ready    agent   26d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000e,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000f      Ready    agent   24d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000f,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000g      Ready    agent   24d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000g,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000h      Ready    agent   24d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000h,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000i      Ready    agent   24d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000i,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000j      Ready    agent   24d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000j,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000k      Ready    agent   24d     v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000k,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000l      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000l,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000m      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000m,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000n      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000n,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000o      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000o,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000p      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000p,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000q      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000q,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000r      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000r,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000s      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000s,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000t      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000t,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbrepool-17134697-vmss00000u      Ready    agent   7d11h   v1.26.3   agentpool=tbrepool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v5,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbrepool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=d33d4970-25d7-11ee-a61f-1a2b57d69b60,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202307.04.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbrepool-17134697-vmss00000u,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v5,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000000   Ready    agent   24d     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000000,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000001   Ready    agent   24d     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000001,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000002   Ready    agent   24d     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000002,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000003   Ready    agent   24d     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000003,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000004   Ready    agent   24d     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000004,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000005   Ready    agent   24d     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000005,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000006   Ready    agent   24h     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000006,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000007   Ready    agent   24h     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000007,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000008   Ready    agent   24h     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000008,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss000009   Ready    agent   21h     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss000009,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss00000a   Ready    agent   21h     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss00000a,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
aks-tbtranspool-11729786-vmss00000b   Ready    agent   21h     v1.26.3   agentpool=tbtranspool,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D4s_v4,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=southeastasia,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/agentpool=tbtranspool,kubernetes.azure.com/cluster=rg-stg-mc-aks-sea,kubernetes.azure.com/consolidated-additional-properties=2b8ca7ed-47ad-11ee-a156-6e02b5896633,kubernetes.azure.com/kubelet-identity-client-id=d455bec7-63d6-4ddb-a2cf-27aff47a1894,kubernetes.azure.com/mode=user,kubernetes.azure.com/node-image-version=AKSUbuntu-2204gen2containerd-202308.10.0,kubernetes.azure.com/nodepool-type=VirtualMachineScaleSets,kubernetes.azure.com/os-sku=Ubuntu,kubernetes.azure.com/role=agent,kubernetes.azure.com/storageprofile=managed,kubernetes.azure.com/storagetier=Premium_LRS,kubernetes.io/arch=amd64,kubernetes.io/hostname=aks-tbtranspool-11729786-vmss00000b,kubernetes.io/os=linux,kubernetes.io/role=agent,node-role.kubernetes.io/agent=,node.kubernetes.io/instance-type=Standard_D4s_v4,storageprofile=managed,storagetier=Premium_LRS,topology.disk.csi.azure.com/zone=,topology.kubernetes.io/region=southeastasia,topology.kubernetes.io/zone=0
```
* Deployment example
[Thingsboard Helm Chart](https://github.com/chienfuchen32/tb-helm-chart)
* Pods in Kubernetes
```bash
kubectl get pod -n thingsboard
NAME                                READY   STATUS    RESTARTS   AGE
kafka-manager-ui-5fb5f96797-2bstg   2/2     Running   0          23d
kafka-ui-7d9f5df786-qv5sj           1/1     Running   0          23d
tb-core-0                           1/1     Running   0          21h
tb-core-1                           1/1     Running   0          21h
tb-core-2                           1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-57zhd     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-5tcws     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-5xlsk     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-6hszn     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-6xv9x     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-b6pb7     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-cbtrk     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-chxwm     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-cnsfs     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-cxnd2     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-kpr6q     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-lfqjj     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-lhzzf     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-ljpn8     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-smkx5     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-stm7c     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-v7cfn     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-vtp4w     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-w52zk     1/1     Running   0          21h
tb-js-executor-7b5f94d5bf-wrnsk     1/1     Running   0          21h
tb-kafka-0                          1/1     Running   0          21h
tb-kafka-1                          1/1     Running   0          21h
tb-kafka-2                          1/1     Running   0          21h
tb-kafka-3                          1/1     Running   0          21h
tb-kafka-4                          1/1     Running   0          21h
tb-kafka-5                          1/1     Running   0          21h
tb-mqtt-transport-0                 1/1     Running   0          20h
tb-mqtt-transport-1                 1/1     Running   0          20h
tb-mqtt-transport-10                1/1     Running   0          20h
tb-mqtt-transport-11                1/1     Running   0          20h
tb-mqtt-transport-2                 1/1     Running   0          20h
tb-mqtt-transport-3                 1/1     Running   0          20h
tb-mqtt-transport-4                 1/1     Running   0          20h
tb-mqtt-transport-5                 1/1     Running   0          20h
tb-mqtt-transport-6                 1/1     Running   0          20h
tb-mqtt-transport-7                 1/1     Running   0          20h
tb-mqtt-transport-8                 1/1     Running   0          20h
tb-mqtt-transport-9                 1/1     Running   0          20h
tb-rule-engine-0                    1/1     Running   0          21h
tb-rule-engine-1                    1/1     Running   0          21h
tb-rule-engine-10                   1/1     Running   0          21h
tb-rule-engine-11                   1/1     Running   0          21h
tb-rule-engine-12                   1/1     Running   0          21h
tb-rule-engine-13                   1/1     Running   0          21h
tb-rule-engine-14                   1/1     Running   0          21h
tb-rule-engine-15                   1/1     Running   0          21h
tb-rule-engine-16                   1/1     Running   0          21h
tb-rule-engine-17                   1/1     Running   0          21h
tb-rule-engine-18                   1/1     Running   0          21h
tb-rule-engine-19                   1/1     Running   0          21h
tb-rule-engine-2                    1/1     Running   0          21h
tb-rule-engine-20                   1/1     Running   0          21h
tb-rule-engine-21                   1/1     Running   0          21h
tb-rule-engine-22                   1/1     Running   0          21h
tb-rule-engine-23                   1/1     Running   0          21h
tb-rule-engine-24                   1/1     Running   0          21h
tb-rule-engine-3                    1/1     Running   0          21h
tb-rule-engine-4                    1/1     Running   0          21h
tb-rule-engine-5                    1/1     Running   0          21h
tb-rule-engine-6                    1/1     Running   0          21h
tb-rule-engine-7                    1/1     Running   0          21h
tb-rule-engine-8                    1/1     Running   0          21h
tb-rule-engine-9                    1/1     Running   0          21h
tb-web-ui-74dd9fbc6-r7ndk           1/1     Running   0          20d
zookeeper-0                         1/1     Running   0          22h
zookeeper-1                         1/1     Running   0          22h
zookeeper-2                         1/1     Running   0          22h
```
* Device Simulater Scenario: 45000 gateway, each gateway prepared 2 end device. Every device generate radomly distributed value for 3 different telemetry key `pulseCounter`, `leakage`, `batteryLevel` per second. Here it comes with [paho-mqtt](https://pypi.org/project/paho-mqtt/) to implement massive clients based on event loop async payload publish to Broker. Expectedly thoughput would be (45000 + 90000 devices) * 3 telemetry / second = 405000 data point per second.
* [perf example](https://github.com/chienfuchen32/thingsboard-flow/blob/main/perf_async.py), [deploy statefulset to K8S](https://github.com/chienfuchen32/thingsboard-flow/blob/main/k8s/perf_async_statefulset.yaml)
## Monitoring
### Kafka
```bash
$ kubectl exec -it -n thingsboard tb-kafka-0 -- bash
# list topics
$ bash-4.4# kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
tb-rule-engine-notifications-node-tb-rule-engine-16
rule-engine-node-tb-rule-engine-4
tb-rule-engine-notifications-node-tb-rule-engine-15
rule-engine-node-tb-rule-engine-5
rule-engine-node-tb-rule-engine-6
tb-rule-engine-notifications-node-tb-rule-engine-18
rule-engine-node-tb-rule-engine-7
tb-rule-engine-notifications-node-tb-rule-engine-17
rule-engine-node-tb-rule-engine-0
rule-engine-node-tb-rule-engine-1
tb-rule-engine-notifications-node-tb-rule-engine-19
rule-engine-node-tb-rule-engine-2
rule-engine-node-tb-rule-engine-3
tb-rule-engine-notifications-node-tb-rule-engine-10
KMOffsetCache-kafka-manager-ui-5fb5f96797-2bstg
tb-rule-engine-notifications-node-tb-rule-engine-12
rule-engine-node-tb-rule-engine-8
rule-engine-node-tb-rule-engine-24
rule-engine-node-tb-rule-engine-9
tb-rule-engine-notifications-node-tb-rule-engine-11
rule-engine-node-tb-rule-engine-22
tb-rule-engine-notifications-node-tb-rule-engine-14
re-Main-consumer
rule-engine-node-tb-rule-engine-23
tb-rule-engine-notifications-node-tb-rule-engine-13
rule-engine-node-tb-rule-engine-20
rule-engine-node-tb-core-2
rule-engine-node-tb-rule-engine-21
rule-engine-node-tb-core-1
rule-engine-node-tb-core-0
rule-engine-node-tb-rule-engine-19
re-HighPriority-consumer
rule-engine-node-tb-rule-engine-17
transport-node-tb-mqtt-transport-11
transport-node-tb-mqtt-transport-0
rule-engine-node-tb-rule-engine-18
transport-node-tb-mqtt-transport-10
tb-rule-engine-notifications-node-tb-rule-engine-21
rule-engine-node-tb-rule-engine-15
tb-rule-engine-notifications-node-tb-rule-engine-20
rule-engine-node-tb-rule-engine-16
tb-rule-engine-notifications-node-tb-rule-engine-23
transport-node-tb-mqtt-transport-3
rule-engine-node-tb-rule-engine-13
transport-node-tb-mqtt-transport-4
rule-engine-node-tb-rule-engine-14
tb-rule-engine-notifications-node-tb-rule-engine-22
rule-engine-node-tb-rule-engine-11
transport-node-tb-mqtt-transport-1
rule-engine-node-tb-rule-engine-12
tb-rule-engine-notifications-node-tb-rule-engine-24
transport-node-tb-mqtt-transport-2
rule-engine-node-tb-rule-engine-10
tb-core-us-consumer
js-executor-group
tb-core-transport-api-consumer
transport-node-tb-mqtt-transport-7
transport-node-tb-mqtt-transport-8
transport-node-tb-mqtt-transport-5
tb-core-node
transport-node-tb-mqtt-transport-6
transport-node-tb-mqtt-transport-9
re-SequentialByOriginator-consumer
tb-core-ota-consumer
tb-rule-engine-notifications-node-tb-rule-engine-8
tb-rule-engine-notifications-node-tb-rule-engine-9
tb-rule-engine-notifications-node-tb-rule-engine-6
tb-rule-engine-notifications-node-tb-rule-engine-7
tb-rule-engine-notifications-node-tb-rule-engine-4
tb-rule-engine-notifications-node-tb-rule-engine-5
tb-rule-engine-notifications-node-tb-rule-engine-2
tb-rule-engine-notifications-node-tb-rule-engine-3
tb-rule-engine-notifications-node-tb-rule-engine-0
tb-core-notifications-node-tb-core-2
tb-rule-engine-notifications-node-tb-rule-engine-1
tb-core-notifications-node-tb-core-1
tb-core-notifications-node-tb-core-0
# check topic configuration
$ bash-4.4# kafka-topics.sh --bootstrap-server localhost:9092 --describe
Topic:tb_rule_engine.notifications.tb-rule-engine-19	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-19	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.notifications.tb-rule-engine-18	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-18	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.notifications.tb-rule-engine-24	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-24	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.notifications.tb-rule-engine-23	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-23	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.notifications.tb-rule-engine-22	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-22	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.notifications.tb-rule-engine-21	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-21	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.main.11	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.11	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.main.11	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.notifications.tb-rule-engine-20	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-20	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.main.10	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.10	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.main.10	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.main.15	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.15	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine.main.15	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.main.14	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.14	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine.main.14	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.main.13	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.13	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.main.13	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.12	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.12	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine.main.12	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.main.19	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.19	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.main.19	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.main.18	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.18	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.main.18	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.17	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.17	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.17	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.16	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.16	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine.main.16	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.notifications.tb-rule-engine-17	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-17	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.notifications.tb-rule-engine-16	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-16	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.notifications.tb-rule-engine-15	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-15	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.notifications.tb-rule-engine-14	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-14	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.notifications.tb-rule-engine-13	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-13	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.notifications.tb-rule-engine-12	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-12	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.notifications.tb-rule-engine-11	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-11	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.notifications.tb-rule-engine-10	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-10	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.main.22	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.22	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.22	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.21	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.21	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.21	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.20	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.20	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.main.20	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.main.26	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.26	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.main.26	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.25	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.25	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.main.25	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.main.24	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.24	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.main.24	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.23	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.23	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.23	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.29	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.29	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine.main.29	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.main.28	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.28	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.28	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.27	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.27	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.main.27	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.main.33	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.33	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.main.33	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.main.32	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.32	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine.main.32	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.notifications.tb-rule-engine-9	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-9	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.31	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.31	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.main.31	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.main.30	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.30	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.30	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.37	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.37	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.main.37	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.36	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.36	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.main.36	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.35	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.35	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.main.35	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.main.34	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.34	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.main.34	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.notifications.tb-rule-engine-2	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-2	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.notifications.tb-rule-engine-1	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-1	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.39	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.39	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.main.39	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.notifications.tb-rule-engine-4	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-4	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.main.38	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.38	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.38	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.notifications.tb-rule-engine-3	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-3	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.notifications.tb-rule-engine-6	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-6	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.notifications.tb-rule-engine-5	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-5	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.notifications.tb-rule-engine-8	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-8	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.notifications.tb-rule-engine-7	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-7	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.notifications.tb-rule-engine-0	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_rule_engine.notifications.tb-rule-engine-0	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.40	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.40	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.40	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_transport.notifications.tb-mqtt-transport-0	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-0	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_transport.notifications.tb-mqtt-transport-1	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-1	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.main.44	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.44	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine.main.44	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_transport.notifications.tb-mqtt-transport-4	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-4	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.main.43	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.43	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine.main.43	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_transport.notifications.tb-mqtt-transport-5	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-5	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.42	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.42	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.main.42	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_transport.notifications.tb-mqtt-transport-2	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-2	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.main.41	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.41	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.main.41	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_transport.notifications.tb-mqtt-transport-3	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-3	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.48	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.48	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.48	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_transport.notifications.tb-mqtt-transport-8	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-8	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.47	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.47	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.47	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_transport.notifications.tb-mqtt-transport-9	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-9	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.46	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.46	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.46	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_transport.notifications.tb-mqtt-transport-6	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-6	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.45	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.45	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.main.45	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_transport.notifications.tb-mqtt-transport-7	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-7	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.main.49	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.49	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.main.49	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.sq.1	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.1	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.sq.1	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.sq.0	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.0	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.sq.0	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.sq.5	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.5	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.sq.5	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.sq.4	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.4	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine.sq.4	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.sq.3	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.3	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.sq.3	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.sq.2	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.2	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.sq.2	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:js_eval.responses.tb-core-0	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-core-0	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:js_eval.responses.tb-core-1	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-core-1	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:js_eval.responses.tb-core-2	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-core-2	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.hp.8	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.8	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.hp.8	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.hp.9	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.9	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.hp.9	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.hp.6	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.6	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.hp.6	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.hp.7	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.7	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine.hp.7	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.hp.4	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.4	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.hp.4	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.hp.5	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.5	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.hp.5	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.hp.2	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.2	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.hp.2	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.hp.3	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.3	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.hp.3	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.hp.0	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.0	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine.hp.0	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.hp.1	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.hp.1	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine.hp.1	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_transport.api.responses.tb-mqtt-transport-3	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-3	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_transport.api.responses.tb-mqtt-transport-2	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-2	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_transport.api.responses.tb-mqtt-transport-1	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-1	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_transport.api.responses.tb-mqtt-transport-0	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-0	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_core.25	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.25	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_core.25	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_core.24	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.24	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_core.24	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_core.27	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.27	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_core.27	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_core.26	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.26	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_core.26	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_core.29	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.29	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_core.29	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_core.28	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.28	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_core.28	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_transport.notifications.tb-mqtt-transport-11	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-11	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_transport.notifications.tb-mqtt-transport-10	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.notifications.tb-mqtt-transport-10	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_core.21	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.21	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_core.21	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_core.20	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.20	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_core.20	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_core.23	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.23	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_core.23	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_core.22	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.22	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_core.22	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_core.14	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.14	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_core.14	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.sq.9	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.9	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.sq.9	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_transport.api.responses.tb-mqtt-transport-7	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-7	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_core.13	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.13	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_core.13	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.sq.8	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.8	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.sq.8	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_transport.api.responses.tb-mqtt-transport-6	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-6	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_core.16	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.16	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_core.16	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.sq.7	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.7	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.sq.7	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_transport.api.responses.tb-mqtt-transport-5	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-5	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_core.15	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.15	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_core.15	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.sq.6	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.sq.6	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.sq.6	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_transport.api.responses.tb-mqtt-transport-4	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-4	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_core.18	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.18	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_core.18	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_core.17	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.17	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_core.17	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_transport.api.responses.tb-mqtt-transport-9	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-9	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_core.19	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.19	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_core.19	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_transport.api.responses.tb-mqtt-transport-8	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-8	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_transport.api.requests	PartitionCount:30	ReplicationFactor:1	Configs:cleanup.policy=delete,segment.bytes=26214400,retention.ms=60000,retention.bytes=104857600
	Topic: tb_transport.api.requests	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_transport.api.requests	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_transport.api.requests	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_transport.api.requests	Partition: 3	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_transport.api.requests	Partition: 4	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_transport.api.requests	Partition: 5	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_transport.api.requests	Partition: 6	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_transport.api.requests	Partition: 7	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_transport.api.requests	Partition: 8	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_transport.api.requests	Partition: 9	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_transport.api.requests	Partition: 10	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_transport.api.requests	Partition: 11	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_transport.api.requests	Partition: 12	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_transport.api.requests	Partition: 13	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_transport.api.requests	Partition: 14	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_transport.api.requests	Partition: 15	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_transport.api.requests	Partition: 16	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_transport.api.requests	Partition: 17	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_transport.api.requests	Partition: 18	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_transport.api.requests	Partition: 19	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_transport.api.requests	Partition: 20	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_transport.api.requests	Partition: 21	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_transport.api.requests	Partition: 22	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_transport.api.requests	Partition: 23	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_transport.api.requests	Partition: 24	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_transport.api.requests	Partition: 25	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_transport.api.requests	Partition: 26	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_transport.api.requests	Partition: 27	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_transport.api.requests	Partition: 28	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_transport.api.requests	Partition: 29	Leader: 3	Replicas: 3	Isr: 3
Topic:__consumer_offsets	PartitionCount:50	ReplicationFactor:1	Configs:compression.type=producer,cleanup.policy=compact,segment.bytes=104857600,retention.ms=300000,retention.bytes=1073741824
	Topic: __consumer_offsets	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: __consumer_offsets	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
	Topic: __consumer_offsets	Partition: 2	Leader: 4	Replicas: 4	Isr: 4
	Topic: __consumer_offsets	Partition: 3	Leader: 5	Replicas: 5	Isr: 5
	Topic: __consumer_offsets	Partition: 4	Leader: 0	Replicas: 0	Isr: 0
	Topic: __consumer_offsets	Partition: 5	Leader: 1	Replicas: 1	Isr: 1
	Topic: __consumer_offsets	Partition: 6	Leader: 2	Replicas: 2	Isr: 2
	Topic: __consumer_offsets	Partition: 7	Leader: 3	Replicas: 3	Isr: 3
	Topic: __consumer_offsets	Partition: 8	Leader: 4	Replicas: 4	Isr: 4
	Topic: __consumer_offsets	Partition: 9	Leader: 5	Replicas: 5	Isr: 5
	Topic: __consumer_offsets	Partition: 10	Leader: 0	Replicas: 0	Isr: 0
	Topic: __consumer_offsets	Partition: 11	Leader: 1	Replicas: 1	Isr: 1
	Topic: __consumer_offsets	Partition: 12	Leader: 2	Replicas: 2	Isr: 2
	Topic: __consumer_offsets	Partition: 13	Leader: 3	Replicas: 3	Isr: 3
	Topic: __consumer_offsets	Partition: 14	Leader: 4	Replicas: 4	Isr: 4
	Topic: __consumer_offsets	Partition: 15	Leader: 5	Replicas: 5	Isr: 5
	Topic: __consumer_offsets	Partition: 16	Leader: 0	Replicas: 0	Isr: 0
	Topic: __consumer_offsets	Partition: 17	Leader: 1	Replicas: 1	Isr: 1
	Topic: __consumer_offsets	Partition: 18	Leader: 2	Replicas: 2	Isr: 2
	Topic: __consumer_offsets	Partition: 19	Leader: 3	Replicas: 3	Isr: 3
	Topic: __consumer_offsets	Partition: 20	Leader: 4	Replicas: 4	Isr: 4
	Topic: __consumer_offsets	Partition: 21	Leader: 5	Replicas: 5	Isr: 5
	Topic: __consumer_offsets	Partition: 22	Leader: 0	Replicas: 0	Isr: 0
	Topic: __consumer_offsets	Partition: 23	Leader: 1	Replicas: 1	Isr: 1
	Topic: __consumer_offsets	Partition: 24	Leader: 2	Replicas: 2	Isr: 2
	Topic: __consumer_offsets	Partition: 25	Leader: 3	Replicas: 3	Isr: 3
	Topic: __consumer_offsets	Partition: 26	Leader: 4	Replicas: 4	Isr: 4
	Topic: __consumer_offsets	Partition: 27	Leader: 5	Replicas: 5	Isr: 5
	Topic: __consumer_offsets	Partition: 28	Leader: 0	Replicas: 0	Isr: 0
	Topic: __consumer_offsets	Partition: 29	Leader: 1	Replicas: 1	Isr: 1
	Topic: __consumer_offsets	Partition: 30	Leader: 2	Replicas: 2	Isr: 2
	Topic: __consumer_offsets	Partition: 31	Leader: 3	Replicas: 3	Isr: 3
	Topic: __consumer_offsets	Partition: 32	Leader: 4	Replicas: 4	Isr: 4
	Topic: __consumer_offsets	Partition: 33	Leader: 5	Replicas: 5	Isr: 5
	Topic: __consumer_offsets	Partition: 34	Leader: 0	Replicas: 0	Isr: 0
	Topic: __consumer_offsets	Partition: 35	Leader: 1	Replicas: 1	Isr: 1
	Topic: __consumer_offsets	Partition: 36	Leader: 2	Replicas: 2	Isr: 2
	Topic: __consumer_offsets	Partition: 37	Leader: 3	Replicas: 3	Isr: 3
	Topic: __consumer_offsets	Partition: 38	Leader: 4	Replicas: 4	Isr: 4
	Topic: __consumer_offsets	Partition: 39	Leader: 5	Replicas: 5	Isr: 5
	Topic: __consumer_offsets	Partition: 40	Leader: 0	Replicas: 0	Isr: 0
	Topic: __consumer_offsets	Partition: 41	Leader: 1	Replicas: 1	Isr: 1
	Topic: __consumer_offsets	Partition: 42	Leader: 2	Replicas: 2	Isr: 2
	Topic: __consumer_offsets	Partition: 43	Leader: 3	Replicas: 3	Isr: 3
	Topic: __consumer_offsets	Partition: 44	Leader: 4	Replicas: 4	Isr: 4
	Topic: __consumer_offsets	Partition: 45	Leader: 5	Replicas: 5	Isr: 5
	Topic: __consumer_offsets	Partition: 46	Leader: 0	Replicas: 0	Isr: 0
	Topic: __consumer_offsets	Partition: 47	Leader: 1	Replicas: 1	Isr: 1
	Topic: __consumer_offsets	Partition: 48	Leader: 2	Replicas: 2	Isr: 2
	Topic: __consumer_offsets	Partition: 49	Leader: 3	Replicas: 3	Isr: 3
Topic:js_eval.responses.tb-rule-engine-12	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-12	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_usage_stats.23	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.23	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_usage_stats.23	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:js_eval.responses.tb-rule-engine-13	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-13	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_usage_stats.22	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.22	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_usage_stats.22	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:js_eval.responses.tb-rule-engine-10	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-10	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_usage_stats.21	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.21	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_usage_stats.21	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:js_eval.responses.tb-rule-engine-11	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-11	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_usage_stats.20	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.20	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_usage_stats.20	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:js_eval.responses.tb-rule-engine-16	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-16	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:js_eval.responses.tb-rule-engine-17	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-17	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:js_eval.responses.tb-rule-engine-14	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-14	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:js_eval.responses.tb-rule-engine-15	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-15	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:js_eval.responses.tb-rule-engine-18	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-18	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:js_eval.responses.tb-rule-engine-19	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-19	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_core.10	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.10	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_core.10	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_core.12	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.12	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_core.12	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_core.11	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.11	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_core.11	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_rule_engine.main.9	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.9	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine.main.9	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.main.7	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.7	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine.main.7	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_rule_engine.main.8	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.8	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.8	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.5	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.5	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine.main.5	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_rule_engine.main.6	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.6	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine.main.6	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.main.3	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.3	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.main.3	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine.main.4	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.4	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine.main.4	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.1	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.1	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine.main.1	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_rule_engine.main.2	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.2	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine.main.2	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_usage_stats.19	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.19	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_usage_stats.19	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_usage_stats.18	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.18	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_usage_stats.18	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_rule_engine.main.0	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_rule_engine.main.0	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine.main.0	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_usage_stats.17	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.17	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_usage_stats.17	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_usage_stats.16	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.16	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_usage_stats.16	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:js_eval.responses.tb-rule-engine-20	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-20	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_usage_stats.15	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.15	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_usage_stats.15	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
Topic:tb_usage_stats.14	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.14	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_usage_stats.14	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_usage_stats.13	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.13	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_usage_stats.13	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:js_eval.responses.tb-rule-engine-23	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-23	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_usage_stats.12	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.12	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_usage_stats.12	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:js_eval.responses.tb-rule-engine-24	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-24	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_usage_stats.11	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.11	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_usage_stats.11	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:js_eval.responses.tb-rule-engine-21	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-21	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_usage_stats.10	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.10	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_usage_stats.10	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:js_eval.responses.tb-rule-engine-22	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-22	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_transport.api.responses.tb-mqtt-transport-10	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-10	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_transport.api.responses.tb-mqtt-transport-11	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_transport.api.responses.tb-mqtt-transport-11	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_rule_engine	PartitionCount:30	ReplicationFactor:1	Configs:cleanup.policy=delete,segment.bytes=26214400,retention.ms=60000,retention.bytes=104857600
	Topic: tb_rule_engine	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine	Partition: 1	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine	Partition: 2	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine	Partition: 3	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine	Partition: 4	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine	Partition: 5	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine	Partition: 6	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine	Partition: 7	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine	Partition: 8	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine	Partition: 9	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine	Partition: 10	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine	Partition: 11	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine	Partition: 12	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine	Partition: 13	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine	Partition: 14	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine	Partition: 15	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine	Partition: 16	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine	Partition: 17	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine	Partition: 18	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine	Partition: 19	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine	Partition: 20	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine	Partition: 21	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine	Partition: 22	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine	Partition: 23	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_rule_engine	Partition: 24	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_rule_engine	Partition: 25	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_rule_engine	Partition: 26	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_rule_engine	Partition: 27	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_rule_engine	Partition: 28	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_rule_engine	Partition: 29	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_usage_stats.2	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.2	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_usage_stats.2	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_usage_stats.3	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.3	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_usage_stats.3	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_usage_stats.4	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.4	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_usage_stats.4	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_usage_stats.5	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.5	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_usage_stats.5	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:js_eval.requests	PartitionCount:100	ReplicationFactor:1	Configs:cleanup.policy=delete,segment.bytes=26214400,retention.ms=60000,retention.bytes=104857600
	Topic: js_eval.requests	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 3	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 4	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 5	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 6	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 7	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 8	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 9	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 10	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 11	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 12	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 13	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 14	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 15	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 16	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 17	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 18	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 19	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 20	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 21	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 22	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 23	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 24	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 25	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 26	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 27	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 28	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 29	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 30	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 31	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 32	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 33	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 34	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 35	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 36	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 37	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 38	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 39	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 40	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 41	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 42	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 43	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 44	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 45	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 46	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 47	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 48	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 49	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 50	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 51	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 52	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 53	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 54	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 55	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 56	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 57	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 58	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 59	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 60	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 61	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 62	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 63	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 64	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 65	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 66	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 67	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 68	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 69	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 70	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 71	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 72	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 73	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 74	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 75	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 76	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 77	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 78	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 79	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 80	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 81	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 82	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 83	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 84	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 85	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 86	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 87	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 88	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 89	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 90	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 91	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 92	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 93	Leader: 1	Replicas: 1	Isr: 1
	Topic: js_eval.requests	Partition: 94	Leader: 2	Replicas: 2	Isr: 2
	Topic: js_eval.requests	Partition: 95	Leader: 3	Replicas: 3	Isr: 3
	Topic: js_eval.requests	Partition: 96	Leader: 4	Replicas: 4	Isr: 4
	Topic: js_eval.requests	Partition: 97	Leader: 5	Replicas: 5	Isr: 5
	Topic: js_eval.requests	Partition: 98	Leader: 0	Replicas: 0	Isr: 0
	Topic: js_eval.requests	Partition: 99	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_usage_stats.0	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.0	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_usage_stats.0	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_usage_stats.1	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.1	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_usage_stats.1	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_usage_stats.6	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.6	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_usage_stats.6	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_usage_stats.7	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.7	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_usage_stats.7	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_usage_stats.8	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.8	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_usage_stats.8	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_usage_stats.9	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.9	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_usage_stats.9	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:js_eval.responses.tb-rule-engine-4	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-4	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:js_eval.responses.tb-rule-engine-3	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-3	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:js_eval.responses.tb-rule-engine-2	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-2	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:js_eval.responses.tb-rule-engine-1	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-1	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:js_eval.responses.tb-rule-engine-8	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-8	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
Topic:js_eval.responses.tb-rule-engine-7	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-7	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:js_eval.responses.tb-rule-engine-6	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-6	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
Topic:js_eval.responses.tb-rule-engine-5	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-5	Partition: 0	Leader: 2	Replicas: 2	Isr: 2
Topic:js_eval.responses.tb-rule-engine-9	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-9	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:js_eval.responses.tb-rule-engine-0	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=104857600
	Topic: js_eval.responses.tb-rule-engine-0	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_core.4	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.4	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_core.4	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_core.5	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.5	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_core.5	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_core.2	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.2	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_core.2	Partition: 1	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_core.3	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.3	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_core.3	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_core.0	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.0	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_core.0	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_core.1	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.1	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_core.1	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_core.notifications.tb-core-0	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_core.notifications.tb-core-0	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
Topic:tb_core.notifications.tb-core-1	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_core.notifications.tb-core-1	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_usage_stats.29	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.29	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_usage_stats.29	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_core.notifications.tb-core-2	PartitionCount:1	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_core.notifications.tb-core-2	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_usage_stats.28	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.28	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_usage_stats.28	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_core.8	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.8	Partition: 0	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_core.8	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_usage_stats.27	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.27	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_usage_stats.27	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
Topic:tb_core.9	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.9	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_core.9	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_usage_stats.26	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.26	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_usage_stats.26	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_core.6	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.6	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_core.6	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_usage_stats.25	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.25	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_usage_stats.25	Partition: 1	Leader: 5	Replicas: 5	Isr: 5
Topic:tb_core.7	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_core.7	Partition: 0	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_core.7	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
Topic:tb_usage_stats.24	PartitionCount:2	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=259200000,retention.bytes=1048576000
	Topic: tb_usage_stats.24	Partition: 0	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_usage_stats.24	Partition: 1	Leader: 2	Replicas: 2	Isr: 2
Topic:tb_ota_package	PartitionCount:10	ReplicationFactor:1	Configs:min.insync.replicas=1,cleanup.policy=delete,segment.bytes=26214400,retention.ms=604800000,retention.bytes=1048576000
	Topic: tb_ota_package	Partition: 0	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_ota_package	Partition: 1	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_ota_package	Partition: 2	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_ota_package	Partition: 3	Leader: 3	Replicas: 3	Isr: 3
	Topic: tb_ota_package	Partition: 4	Leader: 0	Replicas: 0	Isr: 0
	Topic: tb_ota_package	Partition: 5	Leader: 5	Replicas: 5	Isr: 5
	Topic: tb_ota_package	Partition: 6	Leader: 1	Replicas: 1	Isr: 1
	Topic: tb_ota_package	Partition: 7	Leader: 4	Replicas: 4	Isr: 4
	Topic: tb_ota_package	Partition: 8	Leader: 2	Replicas: 2	Isr: 2
	Topic: tb_ota_package	Partition: 9	Leader: 3	Replicas: 3	Isr: 3
# check consumer group
$ bash-4.4# kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group rule-engine-node-tb-rule-engine-5 --describe

TOPIC                              PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                              HOST            CLIENT-ID
js_eval.responses.tb-rule-engine-5 0          0               0               0               js-tb-rule-engine-5-9f25e5dc-7213-4ea7-8b62-4258c6afac30 /10.224.7.222   js-tb-rule-engine-5
# check consumer group
$ bash-4.4# kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group re-Main-consumer --describe

TOPIC                  PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                                                HOST            CLIENT-ID
tb_rule_engine.main.24 0          116024174       116024213       39              re-Main-consumer-tb-rule-engine-9-45-5157f64c-b83a-43c0-940f-216b7b8d51e6  /10.224.20.110  re-Main-consumer-tb-rule-engine-9-45
tb_rule_engine.main.24 1          116013525       116013571       46              re-Main-consumer-tb-rule-engine-9-45-5157f64c-b83a-43c0-940f-216b7b8d51e6  /10.224.20.110  re-Main-consumer-tb-rule-engine-9-45
tb_rule_engine.main.44 0          121386793       121386858       65              re-Main-consumer-tb-rule-engine-4-91-d074e1fa-46fa-4fd3-92d5-7ff295899966  /10.224.6.165   re-Main-consumer-tb-rule-engine-4-91
tb_rule_engine.main.44 1          121385294       121385380       86              re-Main-consumer-tb-rule-engine-4-91-d074e1fa-46fa-4fd3-92d5-7ff295899966  /10.224.6.165   re-Main-consumer-tb-rule-engine-4-91
tb_rule_engine.main.12 0          120495719       120495795       76              re-Main-consumer-tb-rule-engine-2-117-33329166-99c0-4e9c-a452-f0bdb17c097f /10.224.18.154  re-Main-consumer-tb-rule-engine-2-117
tb_rule_engine.main.12 1          120507233       120507310       77              re-Main-consumer-tb-rule-engine-2-117-33329166-99c0-4e9c-a452-f0bdb17c097f /10.224.18.154  re-Main-consumer-tb-rule-engine-2-117
tb_rule_engine.main.5  0          117238448       117238518       70              re-Main-consumer-tb-rule-engine-13-2-16bbacfb-f30d-4adf-9e50-d44e03b09413  /10.224.18.92   re-Main-consumer-tb-rule-engine-13-2
tb_rule_engine.main.5  1          117245577       117245663       86              re-Main-consumer-tb-rule-engine-13-2-16bbacfb-f30d-4adf-9e50-d44e03b09413  /10.224.18.92   re-Main-consumer-tb-rule-engine-13-2
tb_rule_engine.main.4  0          116786386       116786490       104             re-Main-consumer-tb-rule-engine-12-3-aec3fd34-feb8-4694-bdea-4098aac5e1ba  /10.224.13.90   re-Main-consumer-tb-rule-engine-12-3
tb_rule_engine.main.4  1          116791814       116791921       107             re-Main-consumer-tb-rule-engine-12-3-aec3fd34-feb8-4694-bdea-4098aac5e1ba  /10.224.13.90   re-Main-consumer-tb-rule-engine-12-3
tb_rule_engine.main.22 0          118426316       118426417       101             re-Main-consumer-tb-rule-engine-7-59-f17da1ac-004e-47d1-ae39-642c709385c2  /10.224.14.154  re-Main-consumer-tb-rule-engine-7-59
tb_rule_engine.main.22 1          118424054       118424162       108             re-Main-consumer-tb-rule-engine-7-59-f17da1ac-004e-47d1-ae39-642c709385c2  /10.224.14.154  re-Main-consumer-tb-rule-engine-7-59
tb_rule_engine.main.40 0          119083516       119083613       97              re-Main-consumer-tb-rule-engine-22-4-b4fbab75-8302-4730-bb83-31261c0747e8  /10.224.13.245  re-Main-consumer-tb-rule-engine-22-4
tb_rule_engine.main.40 1          119056369       119056465       96              re-Main-consumer-tb-rule-engine-22-4-b4fbab75-8302-4730-bb83-31261c0747e8  /10.224.13.245  re-Main-consumer-tb-rule-engine-22-4
tb_rule_engine.main.20 0          114379730       114379849       119             re-Main-consumer-tb-rule-engine-5-77-62300ec0-87ad-40b9-a4ed-70f234e59ca9  /10.224.7.222   re-Main-consumer-tb-rule-engine-5-77
tb_rule_engine.main.20 1          114392952       114393067       115             re-Main-consumer-tb-rule-engine-5-77-62300ec0-87ad-40b9-a4ed-70f234e59ca9  /10.224.7.222   re-Main-consumer-tb-rule-engine-5-77
tb_rule_engine.main.38 0          117546825       117546929       104             re-Main-consumer-tb-rule-engine-20-6-f9aaed2f-8fc2-4048-b17c-3002dd0b3d03  /10.224.13.147  re-Main-consumer-tb-rule-engine-20-6
tb_rule_engine.main.38 1          117540201       117540323       122             re-Main-consumer-tb-rule-engine-20-6-f9aaed2f-8fc2-4048-b17c-3002dd0b3d03  /10.224.13.147  re-Main-consumer-tb-rule-engine-20-6
tb_rule_engine.main.17 0          121874233       121874372       139             re-Main-consumer-tb-rule-engine-24-1-4377bf95-3278-4588-b441-33d4bc156339  /10.224.2.67    re-Main-consumer-tb-rule-engine-24-1
tb_rule_engine.main.17 1          121873470       121873594       124             re-Main-consumer-tb-rule-engine-24-1-4377bf95-3278-4588-b441-33d4bc156339  /10.224.2.67    re-Main-consumer-tb-rule-engine-24-1
tb_rule_engine.main.35 0          116465335       116465444       109             re-Main-consumer-tb-rule-engine-18-9-204d013f-b698-4f8d-9087-2ad2caeeaae6  /10.224.8.41    re-Main-consumer-tb-rule-engine-18-9
tb_rule_engine.main.35 1          116462786       116462921       135             re-Main-consumer-tb-rule-engine-18-9-204d013f-b698-4f8d-9087-2ad2caeeaae6  /10.224.8.41    re-Main-consumer-tb-rule-engine-18-9
tb_rule_engine.main.34 0          115369704       115369825       121             re-Main-consumer-tb-rule-engine-17-14-54969464-1e01-4a6d-a9e3-4787075ed732 /10.224.11.200  re-Main-consumer-tb-rule-engine-17-14
tb_rule_engine.main.34 1          115331022       115331142       120             re-Main-consumer-tb-rule-engine-17-14-54969464-1e01-4a6d-a9e3-4787075ed732 /10.224.11.200  re-Main-consumer-tb-rule-engine-17-14
tb_rule_engine.main.15 0          116094834       116094947       113             re-Main-consumer-tb-rule-engine-22-2-7f859432-0b99-493f-ae9e-424148f921a2  /10.224.13.245  re-Main-consumer-tb-rule-engine-22-2
tb_rule_engine.main.15 1          116094939       116095068       129             re-Main-consumer-tb-rule-engine-22-2-7f859432-0b99-493f-ae9e-424148f921a2  /10.224.13.245  re-Main-consumer-tb-rule-engine-22-2
tb_rule_engine.main.43 0          117004575       117004706       131             re-Main-consumer-tb-rule-engine-3-107-d3153d1a-f779-4d1b-bd8f-967616d06efc /10.224.21.200  re-Main-consumer-tb-rule-engine-3-107
tb_rule_engine.main.43 1          116999932       117000065       133             re-Main-consumer-tb-rule-engine-3-107-d3153d1a-f779-4d1b-bd8f-967616d06efc /10.224.21.200  re-Main-consumer-tb-rule-engine-3-107
tb_rule_engine.main.6  0          117542251       117542385       134             re-Main-consumer-tb-rule-engine-14-1-85d2216b-5769-47ec-9dab-5ed8f35db86e  /10.224.11.68   re-Main-consumer-tb-rule-engine-14-1
tb_rule_engine.main.6  1          117551091       117551227       136             re-Main-consumer-tb-rule-engine-14-1-85d2216b-5769-47ec-9dab-5ed8f35db86e  /10.224.11.68   re-Main-consumer-tb-rule-engine-14-1
tb_rule_engine.main.7  0          120106886       120107047       161             re-Main-consumer-tb-rule-engine-15-2-1f1288f1-abf7-41a9-ba53-db81ac0842a8  /10.224.19.245  re-Main-consumer-tb-rule-engine-15-2
tb_rule_engine.main.7  1          120133749       120133907       158             re-Main-consumer-tb-rule-engine-15-2-1f1288f1-abf7-41a9-ba53-db81ac0842a8  /10.224.19.245  re-Main-consumer-tb-rule-engine-15-2
tb_rule_engine.main.41 0          115987681       115987859       178             re-Main-consumer-tb-rule-engine-23-3-a64b5784-18fd-436a-b099-7f457ad93ccf  /10.224.7.123   re-Main-consumer-tb-rule-engine-23-3
tb_rule_engine.main.41 1          115987402       115987552       150             re-Main-consumer-tb-rule-engine-23-3-a64b5784-18fd-436a-b099-7f457ad93ccf  /10.224.7.123   re-Main-consumer-tb-rule-engine-23-3
tb_rule_engine.main.25 0          116121506       116121662       156             re-Main-consumer-tb-rule-engine-0-176-cd0f0f8e-1757-4ef2-851b-6dd2607d0889 /10.224.2.132   re-Main-consumer-tb-rule-engine-0-176
tb_rule_engine.main.25 1          116134793       116134959       166             re-Main-consumer-tb-rule-engine-0-176-cd0f0f8e-1757-4ef2-851b-6dd2607d0889 /10.224.2.132   re-Main-consumer-tb-rule-engine-0-176
tb_rule_engine.main.30 0          115993629       115993798       169             re-Main-consumer-tb-rule-engine-13-25-e67380c7-05b7-4d24-9067-c4eda0bc1781 /10.224.18.92   re-Main-consumer-tb-rule-engine-13-25
tb_rule_engine.main.30 1          116003984       116004147       163             re-Main-consumer-tb-rule-engine-13-25-e67380c7-05b7-4d24-9067-c4eda0bc1781 /10.224.18.92   re-Main-consumer-tb-rule-engine-13-25
tb_rule_engine.main.10 0          116248087       116248234       147             re-Main-consumer-tb-rule-engine-18-2-2d251452-f757-4cb8-a17f-7c7cc367931b  /10.224.8.41    re-Main-consumer-tb-rule-engine-18-2
tb_rule_engine.main.10 1          116259777       116259961       184             re-Main-consumer-tb-rule-engine-18-2-2d251452-f757-4cb8-a17f-7c7cc367931b  /10.224.8.41    re-Main-consumer-tb-rule-engine-18-2
tb_rule_engine.main.18 0          117456965       117457198       233             re-Main-consumer-tb-rule-engine-3-106-e5c8cb0e-b909-4501-aa57-964bc79c6eb9 /10.224.21.200  re-Main-consumer-tb-rule-engine-3-106
tb_rule_engine.main.18 1          117452287       117452456       169             re-Main-consumer-tb-rule-engine-3-106-e5c8cb0e-b909-4501-aa57-964bc79c6eb9 /10.224.21.200  re-Main-consumer-tb-rule-engine-3-106
tb_rule_engine.main.11 0          118200513       118200703       190             re-Main-consumer-tb-rule-engine-19-1-f475cc04-2ea2-4c5f-8daa-a3d61a1d1489  /10.224.17.105  re-Main-consumer-tb-rule-engine-19-1
tb_rule_engine.main.11 1          118193920       118194101       181             re-Main-consumer-tb-rule-engine-19-1-f475cc04-2ea2-4c5f-8daa-a3d61a1d1489  /10.224.17.105  re-Main-consumer-tb-rule-engine-19-1
tb_rule_engine.main.46 0          116539903       116540074       171             re-Main-consumer-tb-rule-engine-6-68-6085e78b-b9d4-4e75-80c6-3b5f66577ce8  /10.224.17.132  re-Main-consumer-tb-rule-engine-6-68
tb_rule_engine.main.46 1          116555524       116555724       200             re-Main-consumer-tb-rule-engine-6-68-6085e78b-b9d4-4e75-80c6-3b5f66577ce8  /10.224.17.132  re-Main-consumer-tb-rule-engine-6-68
tb_rule_engine.main.0  0          118928341       118928547       206             re-Main-consumer-tb-rule-engine-0-42-7965c608-e55c-4ec8-9806-6ba198e0a26f  /10.224.2.132   re-Main-consumer-tb-rule-engine-0-42
tb_rule_engine.main.0  1          118946689       118946889       200             re-Main-consumer-tb-rule-engine-0-42-7965c608-e55c-4ec8-9806-6ba198e0a26f  /10.224.2.132   re-Main-consumer-tb-rule-engine-0-42
tb_rule_engine.main.8  0          117504571       117504772       201             re-Main-consumer-tb-rule-engine-16-1-c761f42d-fda5-4a8e-a3bd-34a2eac63c5b  /10.224.17.233  re-Main-consumer-tb-rule-engine-16-1
tb_rule_engine.main.8  1          117521280       117521473       193             re-Main-consumer-tb-rule-engine-16-1-c761f42d-fda5-4a8e-a3bd-34a2eac63c5b  /10.224.17.233  re-Main-consumer-tb-rule-engine-16-1
tb_rule_engine.main.26 0          116562390       116562588       198             re-Main-consumer-tb-rule-engine-1-136-bd66d982-2756-4a54-9002-a08bfdc16dc5 /10.224.12.43   re-Main-consumer-tb-rule-engine-1-136
tb_rule_engine.main.26 1          116559045       116559253       208             re-Main-consumer-tb-rule-engine-1-136-bd66d982-2756-4a54-9002-a08bfdc16dc5 /10.224.12.43   re-Main-consumer-tb-rule-engine-1-136
tb_rule_engine.main.39 0          119167358       119167576       218             re-Main-consumer-tb-rule-engine-21-5-adfdc110-bdf2-4906-ad94-9741102617b5  /10.224.1.211   re-Main-consumer-tb-rule-engine-21-5
tb_rule_engine.main.39 1          119172698       119172909       211             re-Main-consumer-tb-rule-engine-21-5-adfdc110-bdf2-4906-ad94-9741102617b5  /10.224.1.211   re-Main-consumer-tb-rule-engine-21-5
tb_rule_engine.main.45 0          116072636       116072853       217             re-Main-consumer-tb-rule-engine-5-76-40039470-c648-4d04-857f-7a8cb57ed114  /10.224.7.222   re-Main-consumer-tb-rule-engine-5-76
tb_rule_engine.main.45 1          116081785       116082003       218             re-Main-consumer-tb-rule-engine-5-76-40039470-c648-4d04-857f-7a8cb57ed114  /10.224.7.222   re-Main-consumer-tb-rule-engine-5-76
tb_rule_engine.main.2  0          118807357       118807566       209             re-Main-consumer-tb-rule-engine-10-1-79fe77eb-79ff-4bcd-b18a-0017a658d2b6  /10.224.12.209  re-Main-consumer-tb-rule-engine-10-1
tb_rule_engine.main.2  1          118798306       118798529       223             re-Main-consumer-tb-rule-engine-10-1-79fe77eb-79ff-4bcd-b18a-0017a658d2b6  /10.224.12.209  re-Main-consumer-tb-rule-engine-10-1
tb_rule_engine.main.21 0          117598740       117598959       219             re-Main-consumer-tb-rule-engine-6-67-9a3e181e-6111-4364-b9e8-078f18b616e0  /10.224.17.132  re-Main-consumer-tb-rule-engine-6-67
tb_rule_engine.main.21 1          117583024       117583232       208             re-Main-consumer-tb-rule-engine-6-67-9a3e181e-6111-4364-b9e8-078f18b616e0  /10.224.17.132  re-Main-consumer-tb-rule-engine-6-67
tb_rule_engine.main.28 0          116935739       116935975       236             re-Main-consumer-tb-rule-engine-11-33-300cb398-b24b-4383-91fa-df4d570dfcd9 /10.224.19.25   re-Main-consumer-tb-rule-engine-11-33
tb_rule_engine.main.28 1          116958982       116959220       238             re-Main-consumer-tb-rule-engine-11-33-300cb398-b24b-4383-91fa-df4d570dfcd9 /10.224.19.25   re-Main-consumer-tb-rule-engine-11-33
tb_rule_engine.main.27 0          119375601       119375808       207             re-Main-consumer-tb-rule-engine-10-37-28525e27-b70f-494e-acd3-003a94a78022 /10.224.12.209  re-Main-consumer-tb-rule-engine-10-37
tb_rule_engine.main.27 1          119378275       119378503       228             re-Main-consumer-tb-rule-engine-10-37-28525e27-b70f-494e-acd3-003a94a78022 /10.224.12.209  re-Main-consumer-tb-rule-engine-10-37
tb_rule_engine.main.19 0          120705658       120705903       245             re-Main-consumer-tb-rule-engine-4-90-0d26afd1-756c-45d3-b72f-68049d18b873  /10.224.6.165   re-Main-consumer-tb-rule-engine-4-90
tb_rule_engine.main.19 1          120677681       120677915       234             re-Main-consumer-tb-rule-engine-4-90-0d26afd1-756c-45d3-b72f-68049d18b873  /10.224.6.165   re-Main-consumer-tb-rule-engine-4-90
tb_rule_engine.main.31 0          116983211       116983423       212             re-Main-consumer-tb-rule-engine-14-21-5d3d6ff0-5c83-4c4a-9741-11c308a2e3af /10.224.11.68   re-Main-consumer-tb-rule-engine-14-21
tb_rule_engine.main.31 1          116969597       116969842       245             re-Main-consumer-tb-rule-engine-14-21-5d3d6ff0-5c83-4c4a-9741-11c308a2e3af /10.224.11.68   re-Main-consumer-tb-rule-engine-14-21
tb_rule_engine.main.13 0          114903090       114903340       250             re-Main-consumer-tb-rule-engine-20-2-48f2db10-87cf-4add-b135-7c3e8991d5ce  /10.224.13.147  re-Main-consumer-tb-rule-engine-20-2
tb_rule_engine.main.13 1          114910704       114910930       226             re-Main-consumer-tb-rule-engine-20-2-48f2db10-87cf-4add-b135-7c3e8991d5ce  /10.224.13.147  re-Main-consumer-tb-rule-engine-20-2
tb_rule_engine.main.49 0          116960514       116960759       245             re-Main-consumer-tb-rule-engine-9-44-b4d30d3b-963e-4c32-b06c-a83b18996915  /10.224.20.110  re-Main-consumer-tb-rule-engine-9-44
tb_rule_engine.main.49 1          116971669       116971915       246             re-Main-consumer-tb-rule-engine-9-44-b4d30d3b-963e-4c32-b06c-a83b18996915  /10.224.20.110  re-Main-consumer-tb-rule-engine-9-44
tb_rule_engine.main.48 0          120363621       120363893       272             re-Main-consumer-tb-rule-engine-8-52-ac5fe5cf-2fa0-4816-b384-1652bda92c5a  /10.224.12.125  re-Main-consumer-tb-rule-engine-8-52
tb_rule_engine.main.48 1          120364046       120364311       265             re-Main-consumer-tb-rule-engine-8-52-ac5fe5cf-2fa0-4816-b384-1652bda92c5a  /10.224.12.125  re-Main-consumer-tb-rule-engine-8-52
tb_rule_engine.main.14 0          119650746       119651032       286             re-Main-consumer-tb-rule-engine-21-1-1e1a31b4-c271-47e8-9b80-74707e987481  /10.224.1.211   re-Main-consumer-tb-rule-engine-21-1
tb_rule_engine.main.14 1          119636665       119636935       270             re-Main-consumer-tb-rule-engine-21-1-1e1a31b4-c271-47e8-9b80-74707e987481  /10.224.1.211   re-Main-consumer-tb-rule-engine-21-1
tb_rule_engine.main.37 0          119109164       119109445       281             re-Main-consumer-tb-rule-engine-2-123-23a731c8-b342-4d0f-a473-5daa55f0ed5d /10.224.18.154  re-Main-consumer-tb-rule-engine-2-123
tb_rule_engine.main.37 1          119112086       119112333       247             re-Main-consumer-tb-rule-engine-2-123-23a731c8-b342-4d0f-a473-5daa55f0ed5d /10.224.18.154  re-Main-consumer-tb-rule-engine-2-123
tb_rule_engine.main.9  0          116447305       116447568       263             re-Main-consumer-tb-rule-engine-17-3-904a9014-bddb-4c06-9764-dedd607fc7be  /10.224.11.200  re-Main-consumer-tb-rule-engine-17-3
tb_rule_engine.main.9  1          116438049       116438317       268             re-Main-consumer-tb-rule-engine-17-3-904a9014-bddb-4c06-9764-dedd607fc7be  /10.224.11.200  re-Main-consumer-tb-rule-engine-17-3
tb_rule_engine.main.47 0          118405621       118405912       291             re-Main-consumer-tb-rule-engine-7-60-08d5edf6-a1d0-41a6-b6f9-0c34081e0392  /10.224.14.154  re-Main-consumer-tb-rule-engine-7-60
tb_rule_engine.main.47 1          118396343       118396644       301             re-Main-consumer-tb-rule-engine-7-60-08d5edf6-a1d0-41a6-b6f9-0c34081e0392  /10.224.14.154  re-Main-consumer-tb-rule-engine-7-60
tb_rule_engine.main.29 0          118000974       118001231       257             re-Main-consumer-tb-rule-engine-12-29-0cb153ae-8fc4-4f70-8c37-3a0aa3a1ac54 /10.224.13.90   re-Main-consumer-tb-rule-engine-12-29
tb_rule_engine.main.29 1          118027797       118028089       292             re-Main-consumer-tb-rule-engine-12-29-0cb153ae-8fc4-4f70-8c37-3a0aa3a1ac54 /10.224.13.90   re-Main-consumer-tb-rule-engine-12-29
tb_rule_engine.main.16 0          116504304       116504608       304             re-Main-consumer-tb-rule-engine-23-2-5062c24a-e0fc-4463-90ad-035a1307d49a  /10.224.7.123   re-Main-consumer-tb-rule-engine-23-2
tb_rule_engine.main.16 1          116479586       116479862       276             re-Main-consumer-tb-rule-engine-23-2-5062c24a-e0fc-4463-90ad-035a1307d49a  /10.224.7.123   re-Main-consumer-tb-rule-engine-23-2
tb_rule_engine.main.36 0          119852229       119852529       300             re-Main-consumer-tb-rule-engine-19-7-e24eb824-df36-40cc-ac73-62bb22430c4a  /10.224.17.105  re-Main-consumer-tb-rule-engine-19-7
tb_rule_engine.main.36 1          119868213       119868510       297             re-Main-consumer-tb-rule-engine-19-7-e24eb824-df36-40cc-ac73-62bb22430c4a  /10.224.17.105  re-Main-consumer-tb-rule-engine-19-7
tb_rule_engine.main.42 0          116852790       116853078       288             re-Main-consumer-tb-rule-engine-24-2-45b4d89d-260b-44bc-95ef-c0fa88b981ba  /10.224.2.67    re-Main-consumer-tb-rule-engine-24-2
tb_rule_engine.main.42 1          116864701       116864984       283             re-Main-consumer-tb-rule-engine-24-2-45b4d89d-260b-44bc-95ef-c0fa88b981ba  /10.224.2.67    re-Main-consumer-tb-rule-engine-24-2
tb_rule_engine.main.3  0          120540679       120541005       326             re-Main-consumer-tb-rule-engine-11-4-3e1d3457-fb83-4618-aa12-0b8985343597  /10.224.19.25   re-Main-consumer-tb-rule-engine-11-4
tb_rule_engine.main.3  1          120555163       120555455       292             re-Main-consumer-tb-rule-engine-11-4-3e1d3457-fb83-4618-aa12-0b8985343597  /10.224.19.25   re-Main-consumer-tb-rule-engine-11-4
tb_rule_engine.main.32 0          115161228       115161552       324             re-Main-consumer-tb-rule-engine-15-19-64a13c05-696a-4aa5-8884-36b11dfd7435 /10.224.19.245  re-Main-consumer-tb-rule-engine-15-19
tb_rule_engine.main.32 1          115156850       115157161       311             re-Main-consumer-tb-rule-engine-15-19-64a13c05-696a-4aa5-8884-36b11dfd7435 /10.224.19.245  re-Main-consumer-tb-rule-engine-15-19
tb_rule_engine.main.33 0          116279964       116280278       314             re-Main-consumer-tb-rule-engine-16-16-246ba71a-3a86-40aa-8236-0d4e5d5ad85d /10.224.17.233  re-Main-consumer-tb-rule-engine-16-16
tb_rule_engine.main.33 1          116291663       116291996       333             re-Main-consumer-tb-rule-engine-16-16-246ba71a-3a86-40aa-8236-0d4e5d5ad85d /10.224.17.233  re-Main-consumer-tb-rule-engine-16-16
tb_rule_engine.main.1  0          118976007       118976325       318             re-Main-consumer-tb-rule-engine-1-30-79a6b6ee-033a-44b4-99d2-20279a9397cc  /10.224.12.43   re-Main-consumer-tb-rule-engine-1-30
tb_rule_engine.main.1  1          118997986       118998303       317             re-Main-consumer-tb-rule-engine-1-30-79a6b6ee-033a-44b4-99d2-20279a9397cc  /10.224.12.43   re-Main-consumer-tb-rule-engine-1-30
tb_rule_engine.main.23 0          120230817       120231163       346             re-Main-consumer-tb-rule-engine-8-51-8cf08690-4917-447f-800d-c4176194dd41  /10.224.12.125  re-Main-consumer-tb-rule-engine-8-51
tb_rule_engine.main.23 1          120237198       120237501       303             re-Main-consumer-tb-rule-engine-8-51-8cf08690-4917-447f-800d-c4176194dd41  /10.224.12.125  re-Main-consumer-tb-rule-engine-8-51
```
### Cassandra
```cqlsh
// check if total number of tb-rule-engine save telemetry data published by device every minutes.
cassandra@cqlsh:thingsboard> SELECT COUNT(*) from ts_kv_cf WHERE entity_id=e8e9e7a0-462d-11ee-b28b-2ffdf7208992 AND entity_type='DEVICE' and partition=1693526400000 and key in('pulseCounter','leakage','batteryLevel') AND ts>=1695546000000 AND ts<1695546060000;

 count
-------
   180

(1 rows)

Warnings :
Aggregation query used on multiple partition keys (IN restriction)

```
### Thingsboard Rule Chain 
Message count rule node count each device publish telemetry once, here's a example to save to database every 60 second.
<img src="/tb-perf/tb-rule-chain.png" width="1024">

### Thingsboard Dashboard
This dashboard is according to the data provided from message count rule node which describe every tb-rule-engine JAVA instance processed in a short period of time.
<img src="/tb-perf/tb-message-counter-dashboard.png" width="1024">

### Thingsboard API Usage
API Usage page shows that summary of rule node status, micro service message processing metrics, ...etc.
<img src="/tb-perf/tb-api-usage.png" width="1024">

