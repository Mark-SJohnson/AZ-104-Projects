Azure Hub-Spoke Network Topology
AZ-104 Hands-On Project | Networking Domain


Overview
This project demonstrates core Azure networking skills by building a hub-and-spoke VNet topology with VNet peering, Network Security Groups enforcing traffic isolation between spokes, and Azure Bastion providing secure browser-based VM access with no public IPs exposed.
Domain covered: Implement & Manage Virtual Networking (AZ-104 — 15–20% of exam weight)

What I Built

A hub VNet with a shared services subnet and a dedicated AzureBastionSubnet
Two spoke VNets simulating separate workload environments
VNet peering connecting hub to each spoke with bidirectional connectivity
Network Security Groups restricting cross-spoke traffic and internet inbound
Azure Bastion deployed in the hub for secure RDP/SSH access without public IPs
A test VM in spoke1 with no public IP validated via Bastion connection


Architecture
                        ┌─────────────────────────┐
                        │        vnet-hub          │
                        │      10.0.0.0/16         │
                        │                          │
                        │  ┌─────────────────┐     │
                        │  │   snet-shared   │     │
                        │  │  10.0.1.0/24    │     │
                        │  └─────────────────┘     │
                        │                          │
                        │  ┌─────────────────┐     │
                        │  │AzureBastionSubnet│    │
                        │  │  10.0.2.0/26    │     │
                        │  │   [Bastion]     │     │
                        │  └─────────────────┘     │
                        └────────┬────────┬────────┘
                                 │        │
                    VNet Peering │        │ VNet Peering
                                 │        │
               ┌─────────────────┘        └─────────────────┐
               │                                             │
    ┌──────────▼────────┐                      ┌────────────▼──────┐
    │    vnet-spoke1    │                      │    vnet-spoke2    │
    │   10.1.0.0/16     │                      │   10.2.0.0/16     │
    │                   │                      │                   │
    │ ┌───────────────┐ │                      │ ┌───────────────┐ │
    │ │snet-workload1 │ │                      │ │snet-workload2 │ │
    │ │ 10.1.1.0/24   │ │                      │ │ 10.2.1.0/24   │ │
    │ │  [vm-spoke1]  │ │                      │ └───────────────┘ │
    │ │  [nsg-spoke1] │ │                      │   [nsg-spoke2]    │
    │ └───────────────┘ │                      └───────────────────┘
    └───────────────────┘

    ✅ Spokes cannot communicate directly — all traffic routes through hub
    ✅ No public IPs on VMs — access only via Bastion

Resources Created
ResourceTypePurposevnet-hubVirtual Network (10.0.0.0/16)Central hub networkvnet-spoke1Virtual Network (10.1.0.0/16)Workload 1 environmentvnet-spoke2Virtual Network (10.2.0.0/16)Workload 2 environmentsnet-sharedSubnet (10.0.1.0/24)Shared services in hubAzureBastionSubnetSubnet (10.0.2.0/26)Required subnet for Bastionsnet-workload1Subnet (10.1.1.0/24)Spoke 1 workload subnetsnet-workload2Subnet (10.2.1.0/24)Spoke 2 workload subnethub-to-spoke1VNet PeeringHub ↔ Spoke1 connectionhub-to-spoke2VNet PeeringHub ↔ Spoke2 connectionnsg-spoke1Network Security GroupTraffic rules for spoke1nsg-spoke2Network Security GroupTraffic rules for spoke2bastion-hubAzure Bastion (Basic SKU)Secure VM accesspip-bastion-hubPublic IP (Standard)Required for Bastionvm-spoke1Virtual Machine (B1s)Test VM — no public IP

VNet Address Space Design
VNetAddress SpacePurposevnet-hub10.0.0.0/16Central hub — shared services and Bastionvnet-spoke110.1.0.0/16Spoke — workload environment 1vnet-spoke210.2.0.0/16Spoke — workload environment 2

All three address spaces are non-overlapping — a requirement for VNet peering to work.


NSG Rules
nsg-spoke1
PriorityNameDirectionSourceDestinationPortAction100Allow-RDP-MyIPInboundMy IPAny3389Allow200Deny-Internet-InboundInboundInternetAnyAnyDeny
nsg-spoke2
PriorityNameDirectionSourceDestinationPortAction100Deny-Spoke1-InboundInbound10.1.0.0/16AnyAnyDeny

Spokes cannot communicate directly with each other — spoke2 explicitly denies inbound traffic from spoke1's address range.


VNet Peering Configuration
Peering NameLocal VNetRemote VNetStatushub-to-spoke1vnet-hubvnet-spoke1Connectedspoke1-to-hubvnet-spoke1vnet-hubConnectedhub-to-spoke2vnet-hubvnet-spoke2Connectedspoke2-to-hubvnet-spoke2vnet-hubConnected

VNet peering is non-transitive — spoke1 cannot reach spoke2 directly even though both are peered to the hub.


Validation Results
TestMethodResultVM has no public IPPortal Overview pageNo public IP assigned ✅Bastion RDP connectionConnect via Bastion in portalBrowser-based RDP session opened ✅Hub ↔ Spoke peeringEffective routes on VM NIC10.0.0.0/16 shows as VNet peering ✅Cross-spoke traffic blockedNSG rules on nsg-spoke2Spoke1 range denied inbound ✅No internet inboundNSG rules on nsg-spoke1Internet deny rule confirmed ✅

📸 Screenshots of all validation results available in the /screenshots folder


Steps Completed

 Created resource group rg-hub-network
 Created hub VNet with snet-shared and AzureBastionSubnet
 Created spoke1 VNet with snet-workload1
 Created spoke2 VNet with snet-workload2
 Peered hub to spoke1 with bidirectional connectivity
 Peered hub to spoke2 with bidirectional connectivity
 Created and attached NSG to spoke1 subnet
 Created and attached NSG to spoke2 subnet
 Deployed Azure Bastion (Basic SKU) in hub
 Deployed test VM in spoke1 with no public IP
 Connected to VM via Bastion — no public IP required
 Verified peering via effective routes
 Deleted resource group to complete teardown


Key Concepts Demonstrated
Hub-spoke is non-transitive — spoke1 and spoke2 can each reach the hub but cannot reach each other directly. Enabling spoke-to-spoke communication requires either User Defined Routes (UDRs) pointing through a firewall in the hub, or Azure Virtual Network Manager.
NSG rules are stateful — allowing inbound traffic on a port automatically allows the response traffic outbound. You only need to define one direction per rule.
AzureBastionSubnet naming is strict — the subnet must be named exactly AzureBastionSubnet with no variation. Azure rejects Bastion deployment if the name doesn't match.
Bastion requires a public IP but VMs don't — Bastion acts as a jump host inside Azure, using its own public IP to accept connections from the internet and then connecting to VMs privately over the VNet. VMs stay completely off the public internet.
VNet address spaces must not overlap — peering fails if two VNets share any part of their address range. Always plan address spaces before deployment in real environments.

Technologies Used

Microsoft Azure Portal
Azure Virtual Networks (VNet)
VNet Peering
Network Security Groups (NSG)
Azure Bastion (Basic SKU)
Azure Virtual Machines
Azure CLI


CLI Reference
bash# Create hub VNet
az network vnet create \
  --name vnet-hub \
  --resource-group rg-hub-network \
  --address-prefix 10.0.0.0/16 \
  --subnet-name snet-shared \
  --subnet-prefix 10.0.1.0/24

# Add Bastion subnet to hub
az network vnet subnet create \
  --name AzureBastionSubnet \
  --vnet-name vnet-hub \
  --resource-group rg-hub-network \
  --address-prefix 10.0.2.0/26

# Create spoke VNet
az network vnet create \
  --name vnet-spoke1 \
  --resource-group rg-hub-network \
  --address-prefix 10.1.0.0/16 \
  --subnet-name snet-workload1 \
  --subnet-prefix 10.1.1.0/24

# Create VNet peering hub to spoke1
az network vnet peering create \
  --name hub-to-spoke1 \
  --resource-group rg-hub-network \
  --vnet-name vnet-hub \
  --remote-vnet vnet-spoke1 \
  --allow-vnet-access

# Create NSG
az network nsg create \
  --name nsg-spoke1 \
  --resource-group rg-hub-network

# Add NSG rule
az network nsg rule create \
  --name Deny-Internet-Inbound \
  --nsg-name nsg-spoke1 \
  --resource-group rg-hub-network \
  --priority 200 \
  --direction Inbound \
  --source-address-prefixes Internet \
  --destination-port-ranges '*' \
  --access Deny

# Attach NSG to subnet
az network vnet subnet update \
  --name snet-workload1 \
  --vnet-name vnet-spoke1 \
  --resource-group rg-hub-network \
  --nsg nsg-spoke1

Related AZ-104 Exam Objectives

Configure virtual networks
Configure network security groups
Configure Azure VNet peering
Configure Azure Bastion
Configure name resolution
Configure routing