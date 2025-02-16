# AKS Baseline Network Topology

> Note: This is part of the Azure Kubernetes Service (AKS) Baseline cluster reference implementation. For more information check out the [readme file in the root](../README.md).

## Hub VNet

`CIDR: 10.200.0.0/24`

This regional VNet Hub (Shared) is meant to hold the following subnets:

* [Azure Firewall subnet]
* [Gateway subnet]
* [Azure Bastion subnet], with reference NSG in place

> Note: For more information about this topology, you can read more at [Azure Hub-Spoke topology].

## Spoke VNet

`CIDR: 10.240.0.0/16`

This VNet Spoke is meant to hold the following subnets:

* [AKS System Nodepool] and [AKS User Nodepool] subnet
* [AKS Internal Load Balancer subnet]
* [Azure Application Gateway subnet]
* [Private Link Endpoint subnet]
* All with basic NSGs around each

In the future, this VNet might hold more subnets like [ACI Provider instance] subnets, more [AKS Nodepools subnets], and more.

## Subnet details

| Subnet                                                 | Upgrade Node | Nodes/VMs/Instance | % Seasonal scale out | +Nodes/VMs | Max Ips/Pods per VM/Node | [% Max Surge] | [% Max Unavailable] | +Ips/Pods per VM/Node | Tot. Ips/Pods per VM/Node | [Azure Subnet not assignable Ips factor] | [Private Endpoints] | [Minimum Subnet size] | Scaled Subnet size | [Subnet Mask bits] | Cidr       | Host        | Broadcast     |
|--------------------------------------------------------|--------------|--------------------|------------------|------------|--------------------------|---------------|---------------------|-----------------------|---------------------------|------------------------------------------|---------------------|-----------------------|--------------------|--------------------|----------------|-------------|---------------|
| AKS System and User Nodepool Subnet                    | 2            | 5                  | 200              | 10         | [30]                     | 100           | 0                   | 30                    | 60                        | 5                                        | 0                   | 372                   | 982                | 22                 | 10.240.0.0/22  | 10.240.0.0  | 10.240.3.255  |
| AKS Internal Load Balancer Services Subnet             | 0            | 0                  | 0                | 0          | 5                        | 100           | 100                 | 0                     | 5                         | 5                                        | 0                   | 10                    | 10                 | 28                 | 10.240.4.0/28  | 10.240.4.0  | 10.240.4.15   |
| Private Link Endpoint Subnet                           | 0            | 0                  | 0                | 0          | 0                        | 100           | 100                 | 0                     | 0                         | 5                                        | 2                   | 7                     | 7                  | 28                 | 10.240.4.32/28 | 10.240.4.32 | 10.240.4.47   |
| Azure Application Gateway Subnet                       | 0            | [251]              | 0                | 0          | 0                        | 100           | 100                 | 0                     | 0                         | 5                                        | 0                   | 256                   | 256                | 24                 | 10.240.5.0/24  | 10.240.5.0  | 10.240.5.255  |
| Gateway Subnet (GatewaySubnet)                         | 0            | [27<sup>1</sup>]   | 0                | 0          | 0                        | 100           | 100                 | 0                     | 0                         | 5                                        | 0                   | 32                    | 32                 | 27                 | 10.200.0.64/27 | 10.200.0.64 | 10.200.0.95   |
| Azure Firewall Subnet (AzureFirewallSubnet)            | 0            | [59]               | 0                | 0          | 0                        | 100           | 100                 | 0                     | 0                         | 5                                        | 0                   | 64                    | 64                 | 26                 | 10.200.0.0/26  | 10.200.0.0  | 10.200.0.63   |
| Azure Bastion Subnet (AzureBastionSubnet)              | 0            | [27<sup>2</sup>]   | 0                | 0          | 0                        | 100           | 100                 | 0                     | 0                         | 5                                        | 0                   | 32                    | 32                 | 27                 | 10.200.0.96/27 | 10.200.0.96 | 10.200.0.127  |

## Additional Considerations

1. [AKS System Nodepool] and [AKS User Nodepool] subnet:  Multi-tenant or other advanced workloads may have nodepool isolation requirements that might demand more (and likely smaller) subnets.
2. [AKS Internal Load Balancer subnet]: Multi-tenant, multiple SSL termination rules, single PPE supporting dev/QA/UAT, etc could lead to needing more ingress controllers, but for baseline, we should start with one.
3. [Private Endpoints] subnet: Private Links are created for Azure Container Registry and Azure Key Vault, so these Azure services can be accessed using Private Endpoints within the spoke virtual network. There are multiple [Private Link deployment options]; in this implementation they are deployed to a dedicated subnet within the spoke virtual network.

[27<sup>1</sup>]: https://learn.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#gwsub
[251]: https://learn.microsoft.com/azure/application-gateway/configuration-overview#size-of-the-subnet
[59]: https://learn.microsoft.com/azure/firewall/firewall-faq#does-the-firewall-subnet-size-need-to-change-as-the-service-scales
[27<sup>2</sup>]: https://learn.microsoft.com/azure/bastion/bastion-create-host-portal#createhost
[30]: https://learn.microsoft.com/azure/aks/use-system-pools#system-and-user-node-pools
[% Max Surge]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge
[% Max Unavailable]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable
[Add Ips/Pods]: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment
[Azure Subnet not assignable Ips factor]: https://learn.microsoft.com/azure/virtual-network/virtual-network-ip-addresses-overview-arm#allocation-method-1
[Private Endpoints]: https://learn.microsoft.com/azure/private-link/private-endpoint-overview#private-endpoint-properties
[Minimum Subnet size]: https://learn.microsoft.com/azure/aks/configure-azure-cni#plan-ip-addressing-for-your-cluster
[Subnet Mask bits]: https://learn.microsoft.com/azure/virtual-network/virtual-networks-faq#how-small-and-how-large-can-vnets-and-subnets-be
[Azure Hub-Spoke topology]: https://learn.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke
[Azure Firewall subnet]: https://learn.microsoft.com/azure/firewall/firewall-faq#does-the-firewall-subnet-size-need-to-change-as-the-service-scales
[Gateway subnet]: https://learn.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#gwsub
[Azure Application Gateway subnet]: https://learn.microsoft.com/azure/application-gateway/configuration-infrastructure#virtual-network-and-dedicated-subnet
[Private Link Endpoint subnet]: https://learn.microsoft.com/azure/architecture/guide/networking/private-link-hub-spoke-network#networking
[Private Link deployment options]: https://learn.microsoft.com/azure/architecture/guide/networking/private-link-hub-spoke-network#decision-tree-for-private-link-deployment
[Azure Bastion subnet]: https://learn.microsoft.com/azure/bastion/bastion-create-host-portal#createhost
[AKS System Nodepool]: https://learn.microsoft.com/azure/aks/use-system-pools#system-and-user-node-pools
[AKS User Nodepool]: https://learn.microsoft.com/azure/aks/use-system-pools#system-and-user-node-pools
[AKS Internal Load Balancer subnet]: https://learn.microsoft.com/azure/aks/internal-lb#specify-a-different-subnet
[ACI Provider Instance]: https://learn.microsoft.com/azure/container-instances/container-instances-vnet
[AKS Nodepools subnets]: https://learn.microsoft.com/azure/aks/use-system-pools#system-and-user-node-pools
