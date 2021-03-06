---
title: 'Gewusst wie: Erstellen von NSGs in Azure Resource Manager mit PowerShell | Microsoft Docs'
description: Hier erhalten Sie Informationen zum Erstellen und Bereitstellen von NSGs in Azure Resource Manager mit PowerShell.
services: virtual-network
documentationcenter: na
author: jimdial
manager: carmonm
editor: tysonn
tags: azure-resource-manager

ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/23/2016
ms.author: jdial

---
# <a name="how-to-create-nsgs-in-resource-manager-by-using-powershell"></a>Gewusst wie: Erstellen von NSGs in Resource Manager mit PowerShell
[!INCLUDE [virtual-networks-create-nsg-selectors-arm-include](../../includes/virtual-networks-create-nsg-selectors-arm-include.md)]

[!INCLUDE [virtual-networks-create-nsg-intro-include](../../includes/virtual-networks-create-nsg-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

Dieser Artikel gilt für das Ressourcen-Manager-Bereitstellungsmodell. Sie haben auch die Möglichkeit, [NSGs im klassischen Bereitstellungsmodell zu erstellen](virtual-networks-create-nsg-classic-ps.md).

[!INCLUDE [virtual-networks-create-nsg-scenario-include](../../includes/virtual-networks-create-nsg-scenario-include.md)]

Die folgenden Beispielbefehle für PowerShell setzen voraus, dass bereits eine einfache Umgebung erstellt wurde, die auf dem zuvor beschriebenen Szenario basiert. Wenn Sie die Befehle so ausführen möchten, wie sie in diesem Dokument angezeigt werden, erstellen Sie zunächst die Testumgebung durch Bereitstellen [dieser Vorlage](http://github.com/telmosampaio/azure-templates/tree/master/201-IaaS-WebFrontEnd-SQLBackEnd). Klicken Sie auf **In Azure bereitstellen**, ersetzen Sie bei Bedarf die Standardparameterwerte, und befolgen Sie dann die Anweisungen im Portal.

## <a name="how-to-create-the-nsg-for-the-front-end-subnet"></a>Erstellen der NSG für das Front-End-Subnetz
Führen Sie zum Erstellen einer NSG mit dem Namen *NSG-FrontEnd* basierend auf dem oben beschriebenen Szenario die folgenden Schritte aus:

[!INCLUDE [powershell-preview-include.md](../../includes/powershell-preview-include.md)]

1. Wenn Sie Azure PowerShell zuvor noch nicht verwendet haben, lesen Sie [Installieren und Konfigurieren von Azure PowerShell](../powershell-install-configure.md) , und befolgen Sie die komplette Anleitung, um sich bei Azure anzumelden und Ihr Abonnement auszuwählen.
2. Erstellen Sie eine Sicherheitsregel, die den Zugriff über das Internet auf Port 3389 zulässt.
   
        $rule1 = New-AzureRmNetworkSecurityRuleConfig -Name rdp-rule -Description "Allow RDP" `
            -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 `
            -SourceAddressPrefix Internet -SourcePortRange * `
            -DestinationAddressPrefix * -DestinationPortRange 3389
3. Erstellen Sie eine Sicherheitsregel, die den Zugriff über das Internet auf Port 80 zulässt.
   
        $rule2 = New-AzureRmNetworkSecurityRuleConfig -Name web-rule -Description "Allow HTTP" `
            -Access Allow -Protocol Tcp -Direction Inbound -Priority 101 `
            -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * `
            -DestinationPortRange 80
4. Fügen Sie die oben erstellten Regeln einer neuen NSG mit dem Namen **NSG-FrontEnd**hinzu.
   
        $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName TestRG -Location westus `
        -Name "NSG-FrontEnd" -SecurityRules $rule1,$rule2
5. Überprüfen Sie die in der NSG erstellten Regeln.
   
        $nsg
   
    Die Ausgabe zeigt nur die Sicherheitsregeln:
   
        SecurityRules        : [
                                 {
                                   "Name": "rdp-rule",
                                   "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                                   "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/rdp-rule",
                                   "Description": "Allow RDP",
                                   "Protocol": "Tcp",
                                   "SourcePortRange": "*",
                                   "DestinationPortRange": "3389",
                                   "SourceAddressPrefix": "Internet",
                                   "DestinationAddressPrefix": "*",
                                   "Access": "Allow",
                                   "Priority": 100,
                                   "Direction": "Inbound",
                                   "ProvisioningState": "Succeeded"
                                 },
                                 {
                                   "Name": "web-rule",
                                   "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                                   "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/web-rule",
                                   "Description": "Allow HTTP",
                                   "Protocol": "Tcp",
                                   "SourcePortRange": "*",
                                   "DestinationPortRange": "80",
                                   "SourceAddressPrefix": "Internet",
                                   "DestinationAddressPrefix": "*",
                                   "Access": "Allow",
                                   "Priority": 101,
                                   "Direction": "Inbound",
                                   "ProvisioningState": "Succeeded"
                                 }
                               ]
6. Ordnen Sie die oben erstellte NSG dem Subnetz *FrontEnd* zu.
   
                    $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
                    Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd
                        -AddressPrefix 192.168.1.0/24 -NetworkSecurityGroup $nsg
   
                Output showing only the *FrontEnd* subnet settings, notice the value for the **NetworkSecurityGroup** property:
   
                    Subnets           : [
                                          {
                                            "Name": "FrontEnd",
                                            "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                                            "AddressPrefix": "192.168.1.0/24",
                                            "IpConfigurations": [
                                              {
                                                "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNICWeb2/ipConfigurations/ipconfig1"
                                              },
                                              {
                                                "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1/ipConfigurations/ipconfig1"
                                              }
                                            ],
                                            "NetworkSecurityGroup": {
                                              "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
                                            },
                                            "RouteTable": null,
                                            "ProvisioningState": "Succeeded"
                                          }
   
   > [!WARNING]
   > Die Ausgabe des obigen Befehls zeigt den Inhalt des Konfigurationsobjekts des virtuellen Netzwerks an, das nur auf dem Computer vorhanden ist, auf dem PowerShell ausgeführt wird. Zum Speichern der Änderungen in Azure müssen Sie das Cmdlet `Set-AzureRmVirtualNetwork` ausführen.
   > 
   > 
7. Speichern Sie die neuen Einstellungen für das VNet in Azure.
   
        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
   
    Die Ausgabe zeigt nur den NSG-Teil:
   
        "NetworkSecurityGroup": {
          "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
        }

## <a name="how-to-create-the-nsg-for-the-back-end-subnet"></a>Erstellen der NSG für das Back-End-Subnetz
Führen Sie zum Erstellen einer NSG mit dem Namen *NSG-BackEnd* basierend auf dem oben beschriebenen Szenario die folgenden Schritte aus:

1. Erstellen Sie eine Sicherheitsregel, die den Zugriff aus dem Front-End-Subnetz auf Port 1433 zulässt (von SQL Server verwendeter Standardport).
   
        $rule1 = New-AzureRmNetworkSecurityRuleConfig -Name frontend-rule -Description "Allow FE subnet" `
            -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 `
            -SourceAddressPrefix 192.168.1.0/24 -SourcePortRange * `
            -DestinationAddressPrefix * -DestinationPortRange 1433
2. Erstellen Sie eine Sicherheitsregel, die den Zugriff auf das Internet blockiert.
   
        $rule2 = New-AzureRmNetworkSecurityRuleConfig -Name web-rule -Description "Block Internet" `
            -Access Deny -Protocol * -Direction Outbound -Priority 200 `
            -SourceAddressPrefix * -SourcePortRange * `
            -DestinationAddressPrefix Internet -DestinationPortRange *
3. Fügen Sie die oben erstellten Regeln einer neuen NSG mit dem Namen **NSG-BackEnd**hinzu.
   
        $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName TestRG -Location westus -Name "NSG-BackEnd" `
            -SecurityRules $rule1,$rule2
4. Ordnen Sie die oben erstellte NSG dem Subnetz *BackEnd* zu.
   
        Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name BackEnd
            -AddressPrefix 192.168.2.0/24 -NetworkSecurityGroup $nsg
   
    Die Ausgabe zeigt nur die Einstellungen für das Subnetz *BackEnd*. Beachten Sie den Wert für die Eigenschaft **NetworkSecurityGroup**:
   
        Subnets           : [
                      {
                        "Name": "BackEnd",
                        "Etag": "W/\"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx\"",
                        "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd",
                        "AddressPrefix": "192.168.2.0/24",
                        "IpConfigurations": [...],
                        "NetworkSecurityGroup": {
                          "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd"
                        },
                        "RouteTable": null,
                        "ProvisioningState": "Succeeded"
                      }
5. Speichern Sie die neuen Einstellungen für das VNet in Azure.
   
        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

## <a name="how-to-remove-an-nsg"></a>Vorgehensweise beim Entfernen einer NSG
Um eine vorhandene NSG, in diesem Fall *NSG-Frontend* genannt, zu löschen, führen Sie den folgenden Schritt aus:

Führen Sie **Remove-AzureRmNetworkSecurityGroup** aus wie unten angezeigt, und fügen Sie die Ressourcengruppe hinzu, in der sich die NSG befindet.

            Remove-AzureRmNetworkSecurityGroup -Name "NSG-FrontEnd" -ResourceGroupName "TestRG"



<!--HONumber=Oct16_HO2-->


