---
title: 'Tutorial: Link a VNet to a ExpressRoute circuit - Azure PowerShell'
description: This tutorial provides an overview of how to link virtual networks (VNets) to ExpressRoute circuits by using the Resource Manager deployment model and Azure PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 12/03/2021
ms.author: duau
ms.custom: seodec18, devx-track-azurepowershell

---
# Tutorial: Connect a virtual network to an ExpressRoute circuit
> [!div class="op_single_selector"]
> * [Azure portal](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [Video - Azure portal](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (classic)](expressroute-howto-linkvnet-classic.md)
>

This article helps you link virtual networks (VNets) to Azure ExpressRoute circuits by using the Resource Manager deployment model and PowerShell. Virtual networks can either be in the same subscription or part of another subscription. This article also shows you how to update a virtual network link.

* You can link up to 10 virtual networks to a standard ExpressRoute circuit. All virtual networks must be in the same geopolitical region when using a standard ExpressRoute circuit. 

* A single VNet can be linked to up to 16 ExpressRoute circuits. Use the steps in this article to create a new connection object for each ExpressRoute circuit you're connecting to. The ExpressRoute circuits can be in the same subscription, different subscriptions, or a mix of both.

* If you enable the ExpressRoute premium add-on, you can link virtual networks outside of the geopolitical region of the ExpressRoute circuit. The premium add-on will also allow you to connect more than 10 virtual networks to your ExpressRoute circuit depending on the bandwidth chosen. Check the [FAQ](expressroute-faqs.md) for more details on the premium add-on.

* In order to create the connection from the ExpressRoute circuit to the target ExpressRoute virtual network gateway, the number of address spaces advertised from the local or peered virtual networks needs to be equal to or less than **200**. Once the connection has been successfully created, you can add additional address spaces, up to 1,000, to the local or peered virtual networks.

In this tutorial, you learn how to:
> [!div class="checklist"]
> - Connect a virtual network in the same subscription to a circuit
> - Connect a virtual network in a different subscription to a circuit
> - Modify a virtual network connection
> - Configure ExpressRoute FastPath

## Prerequisites

* Review the [prerequisites](expressroute-prerequisites.md), [routing requirements](expressroute-routing.md), and [workflows](expressroute-workflows.md) before you begin configuration.

* You must have an active ExpressRoute circuit. 
  * Follow the instructions to [create an ExpressRoute circuit](expressroute-howto-circuit-arm.md) and have the circuit enabled by your connectivity provider. 
  * Ensure that you have Azure private peering configured for your circuit. See the [configure routing](expressroute-howto-routing-arm.md) article for routing instructions. 
  * Ensure that Azure private peering gets configured and establishes BGP peering between your network and Microsoft for end-to-end connectivity.
  * Ensure that you have a virtual network and a virtual network gateway created and fully provisioned. Follow the instructions to [create a virtual network gateway for ExpressRoute](expressroute-howto-add-gateway-resource-manager.md). A virtual network gateway for ExpressRoute uses the GatewayType 'ExpressRoute', not VPN.

### Working with Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

## Connect a virtual network in the same subscription to a circuit
You can connect a virtual network gateway to an ExpressRoute circuit by using the following cmdlet. Make sure that the virtual network gateway is created and is ready for linking before you run the cmdlet:

```azurepowershell-interactive
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$gw = Get-AzVirtualNetworkGateway -Name "ExpressRouteGw" -ResourceGroupName "MyRG"
$connection = New-AzVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "MyRG" -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
```

## Connect a virtual network in a different subscription to a circuit
You can share an ExpressRoute circuit across multiple subscriptions. The following figure shows a simple schematic of how sharing works for ExpressRoute circuits across multiple subscriptions.

> [!NOTE]
> Connecting virtual networks between Azure sovereign clouds and Public Azure cloud is not supported. You can only link virtual networks from different subscriptions in the same cloud.

Each of the smaller clouds within the large cloud is used to represent subscriptions that belong to different departments within an organization. Each of the departments within the organization uses their own subscription for deploying their services--but they can share a single ExpressRoute circuit to connect back to your on-premises network. A single department (in this example: IT) can own the ExpressRoute circuit. Other subscriptions within the organization may use the ExpressRoute circuit.

> [!NOTE]
> Connectivity and bandwidth charges for the ExpressRoute circuit will be applied to the subscription owner. All virtual networks share the same bandwidth.
> 

:::image type="content" source="./media/expressroute-howto-linkvnet-classic/cross-subscription.png" alt-text="Cross-subscription connectivity":::

### Administration - circuit owners and circuit users

The 'circuit owner' is an authorized Power User of the ExpressRoute circuit resource. The circuit owner can create authorizations that can be redeemed by 'circuit users'. Circuit users are owners of virtual network gateways that aren't within the same subscription as the ExpressRoute circuit. Circuit users can redeem authorizations (one authorization per virtual network).

The circuit owner has the power to modify and revoke authorizations at any time. Revoking an authorization results in all link connections being deleted from the subscription whose access was revoked.

### Circuit owner operations

**To create an authorization**

The circuit owner creates an authorization, which creates an authorization key to be used by a circuit user to connect their virtual network gateways to the ExpressRoute circuit. An authorization is valid for only one connection.

The following cmdlet snippet shows how to create an authorization:

```azurepowershell-interactive
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
Add-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization1"
Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit

$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$auth1 = Get-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization1"
```

The response to the previous commands will contain the authorization key and status:

```azurepowershell
Name                   : MyAuthorization1
Id                     : /subscriptions/&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/CrossSubTest/authorizations/MyAuthorization1
Etag                   : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&& 
AuthorizationKey       : ####################################
AuthorizationUseStatus : Available
ProvisioningState      : Succeeded
```

**To review authorizations**

The circuit owner can review all authorizations that are issued on a particular circuit by running the following cmdlet:

```azurepowershell-interactive
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$authorizations = Get-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit
```

**To add authorizations**

The circuit owner can add authorizations by using the following cmdlet:

```azurepowershell-interactive
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
Add-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit -Name "MyAuthorization2"
Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit

$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
$authorizations = Get-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $circuit
```

**To delete authorizations**

The circuit owner can revoke/delete authorizations to the user by running the following cmdlet:

```azurepowershell-interactive
Remove-AzExpressRouteCircuitAuthorization -Name "MyAuthorization2" -ExpressRouteCircuit $circuit
Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit
```    

### Circuit user operations

The circuit user needs the peer ID and an authorization key from the circuit owner. The authorization key is a GUID.

Peer ID can be checked from the following command:

```azurepowershell-interactive
Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
```

**To redeem a connection authorization**

The circuit user can run the following cmdlet to redeem a link authorization:

```azurepowershell-interactive
$id = "/subscriptions/********************************/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/MyCircuit"    
$gw = Get-AzVirtualNetworkGateway -Name "ExpressRouteGw" -ResourceGroupName "MyRG"
$connection = New-AzVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "RemoteResourceGroup" -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $id -ConnectionType ExpressRoute -AuthorizationKey "^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^"
```

**To release a connection authorization**

You can release an authorization by deleting the connection that links the ExpressRoute circuit to the virtual network.

## Modify a virtual network connection
You can update certain properties of a virtual network connection. 

**To update the connection weight**

Your virtual network can be connected to multiple ExpressRoute circuits. You may receive the same prefix from more than one ExpressRoute circuit. To choose which connection to send traffic destined for this prefix, you can change *RoutingWeight* of a connection. Traffic will be sent on the connection with the highest *RoutingWeight*.

```azurepowershell-interactive
$connection = Get-AzVirtualNetworkGatewayConnection -Name "MyVirtualNetworkConnection" -ResourceGroupName "MyRG"
$connection.RoutingWeight = 100
Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection
```

The range of *RoutingWeight* is 0 to 32000. The default value is 0.

## Configure ExpressRoute FastPath 
You can enable [ExpressRoute FastPath](expressroute-about-virtual-network-gateways.md) if your virtual network gateway is Ultra Performance or ErGw3AZ. FastPath improves data path performance such as packets per second and connections per second between your on-premises network and your virtual network. 

**Configure FastPath on a new connection**

```azurepowershell-interactive 
$circuit = Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG" 
$gw = Get-AzVirtualNetworkGateway -Name "MyGateway" -ResourceGroupName "MyRG" 
$connection = New-AzVirtualNetworkGatewayConnection -Name "MyConnection" -ResourceGroupName "MyRG" -ExpressRouteGatewayBypass -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute -Location "MyLocation" 
``` 

**Updating an existing connection to enable FastPath**

```azurepowershell-interactive 
$connection = Get-AzVirtualNetworkGatewayConnection -Name "MyConnection" -ResourceGroupName "MyRG" 
$connection.ExpressRouteGatewayBypass = $True
Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection
``` 

### FastPath and Private Link for 100Gbps ExpressRoute Direct

With FastPath and Private Link, Private Link traffic sent over ExpressRoute bypassess the ExpressRoute virtual network gateway in the data path. This is supported for connections associated to 100Gb ExpressRoute Direct circuits. To enable this, follow the below guidance:
1. Send an email to **ERFastPathPL@microsoft.com**, providing the following information: 
* Azure Subscription ID
* Virtual Network (Vnet) Resource ID
* Azure Region where the Private Endpoint/Private Link service is deployed

2. Once you receive a confirmation from Step 1, run the following Azure PowerShell command in the target Azure subscription.
 ```azurepowershell-interactive
Register-AzProviderFeature -FeatureName ExpressRoutePrivateEndpointGatewayBypass -ProviderNamespace Microsoft.Network
```
3. Disable and Enable FastPath on the target connection(s) to enables the changes. Once this step is complete. 100Gb Private Link traffic over ExpressRoute will bypass the ExpressRoute Virtual Network Gateway in the data path.


> [!NOTE]
> You can use [Connection Monitor](how-to-configure-connection-monitor.md) to verify that your traffic is reaching the destination using FastPath.
>

## Enroll in ExpressRoute FastPath features (preview)

FastPath support for virtual network peering is now in Public preview, both IPv4 and IPv6 scenarios are supported. IPv4 FastPath and Vnet peering can be enabled on connections associated to both ExpressRoute Direct and ExpressRoute Partner circuits. IPv6 FastPath and Vnet peering support is limited to connections associated to ExpressRoute Direct.

### FastPath and virtual network peering

With FastPath and virtual network peering, you can enable ExpressRoute connectivity directly to VMs in a local or peered virtual network, bypassing the ExpressRoute virtual network gateway in the data path.

To enroll in this preview, run the follow Azure PowerShell command in the target Azure subscription:

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName ExpressRouteVnetPeeringGatewayBypass -ProviderNamespace Microsoft.Network
```

### FastPath and Private Link for 10Gbps ExpressRoute Direct

With FastPath and Private Link, Private Link traffic sent over ExpressRoute bypassess the ExpressRoute virtual network gateway in the data path. This preview supports connections associated to 10Gbps ExpressRoute Direct circuits. This preview doesn't support ExpressRoute circuits managed by an ExpressRoute partner.

To enroll in this preview, run the following Azure PowerShell command in the target Azure subscription:

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName ExpressRoutePrivateEndpointGatewayBypass -ProviderNamespace Microsoft.Network
```

> [!NOTE]
> Any connections configured for FastPath in the target subscription will be enrolled in the selected preview. We do not advise enabling these previews in production subscriptions.
> If you already have FastPath configured and want to enroll in the preview feature, you need to do the following:
>   1. Enroll in one of the FastPath preview features with the Azure PowerShell commands above.
>   1. Disable and then re-enable FastPath on the target connection.
>   1. To switch between preview features, register the subscription with the target preview PowerShell command, and then disable and re-enable FastPath on the connection.
>

## Clean up resources

If you no longer need the ExpressRoute connection, from the subscription where the gateway is located use the `Remove-AzVirtualNetworkGatewayConnection` command to remove the link between the gateway and the circuit.

```azurepowershell-interactive
Remove-AzVirtualNetworkGatewayConnection "MyConnection" -ResourceGroupName "MyRG"
```

## Next steps
For more information about ExpressRoute, see the ExpressRoute FAQ.

> [!div class="nextstepaction"]
> [ExpressRoute FAQ](expressroute-faqs.md)
