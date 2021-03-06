---
title: Simulierte Hybrid Cloud-Testumgebung | Microsoft Docs
description: Erstellen Sie eine simulierte Hybrid Cloud-Umgebung für Tests durch IT-Spezialisten oder für Entwicklungstests, indem Sie zwei virtuelle Azure-Netzwerke und eine VNet-zu-VNet-Verbindung verwenden.
services: virtual-machines-windows
documentationcenter: ''
author: JoeDavies-MSFT
manager: timlt
editor: ''
tags: azure-resource-manager

ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 08/08/2016
ms.author: josephd

---
# Einrichten einer simulierten Hybrid Cloud-Umgebung zu Testzwecken
In diesem Artikel lernen Sie Schritt für Schritt, wie Sie unter Verwendung von zwei virtuellen Azure-Netzwerken eine simulierte Hybrid Cloud-Umgebung mit Microsoft Azure erstellen. Die resultierende Konfiguration sieht folgendermaßen aus.

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-sim/virtual-machines-windows-ps-hybrid-cloud-test-env-sim-ph4.png)

Dadurch wird eine Hybrid Cloud-Produktionsumgebung mit folgenden Bestandteilen simuliert:

* Einem simulierten und vereinfachten lokalen Netzwerk (TestLab), das in einem virtuellen Azure-Netzwerk gehostet wird
* Einem simulierten standortübergreifenden virtuellen Netzwerk (TestVNET), das in Azure gehostet wird
* Einer VNet-zu-VNet-Verbindung zwischen den beiden virtuellen Netzwerken
* Einem sekundären Domänencontroller im virtuellen Netzwerk TestVNET.

Diese Konfiguration bietet eine Grundlage und einen allgemeinen Ausgangspunkt für Folgendes:

* Entwickeln und Testen von Anwendungen in einer simulierten Hybrid Cloud-Umgebung
* Erstellen von Testkonfigurationen für Computer, einige davon im virtuellen Netzwerk TestLab und einige im virtuellen Netzwerks TestVNET, um IT-Workloads auf Hybrid Cloud-Basis zu simulieren.

Die Einrichtung dieser Hybrid Cloud-Testumgebung besteht aus vier Hauptphasen:

1. Konfigurieren des virtuellen Netzwerks TestLab
2. Erstellen des standortübergreifenden virtuellen Netzwerks TestVNET
3. Herstellen der VNet-zu-VNet-VPN-Verbindung
4. Konfigurieren von DC2

Für diese Konfiguration wird ein Azure-Abonnement benötigt. Wenn Sie ein MSDN- oder Visual Studio-Abonnement besitzen, finden Sie weitere Informationen unter [Monatliche Azure-Gutschrift für Visual Studio-Abonnenten](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/).

> [!NOTE]
> Für virtuelle Computer und virtuelle Netzwerkgateways in Azure fallen laufende Kosten an, wenn sie ausgeführt werden. Diese Kosten werden im Rahmen Ihres MSDN-Abonnements oder Ihres kostenpflichtigen Abonnements abgerechnet. Ein Azure-VPN-Gateway wird als Gruppe von zwei virtuellen Computern in Azure implementiert. Erstellen Sie zum Verringern der Kosten die Testumgebung, und führen Sie die erforderlichen Tests und Demonstrationen möglichst schnell aus.
> 
> 

## Phase 1: Konfigurieren des virtuellen Netzwerks "TestLab"
Erstellen Sie gemäß den Anweisungen des Themas [Testumgebung für die Basiskonfiguration](https://technet.microsoft.com/library/mt771177.aspx) die Computer „DC1“, „APP1“ und „CLIENT1“ im virtuellen Azure-Netzwerk „TestLab“.

Starten Sie als Nächstes eine Azure PowerShell-Eingabeaufforderung.

> [!NOTE]
> Die folgenden Befehlssätze verwenden Azure PowerShell 1.0 und höher.
> 
> 

Melden Sie sich bei Ihrem Konto an.

    Login-AzureRMAccount

Rufen Sie Ihren Abonnementnamen mit dem folgenden Befehl ab.

    Get-AzureRMSubscription | Sort SubscriptionName | Select SubscriptionName

Legen Sie Ihr Azure-Abonnement fest. Verwenden Sie das gleiche Abonnement, das Sie zum Erstellen der grundlegenden Konfiguration (Phase 1) verwendet haben. Ersetzen Sie den Inhalt der Anführungszeichen (einschließlich der Zeichen „<“ und „>“) durch den korrekten Namen.

    $subscr="<subscription name>"
    Get-AzureRmSubscription –SubscriptionName $subscr | Select-AzureRmSubscription

Fügen Sie als Nächstes dem virtuellen Netzwerk „TestLab“ Ihrer Basiskonfiguration ein Gatewaysubnetz zum Hosten des Azure-Gateways hinzu.

    $rgName="<name of your resource group that you used for your TestLab virtual network>"
    $locName="<Azure location name where you placed the TestLab virtual network, such as West US>"
    $vnet=Get-AzureRmVirtualNetwork -ResourceGroupName $rgName -Name TestLab
    Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix 10.255.255.248/29 -VirtualNetwork $vnet
    Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

Fordern Sie anschließend eine öffentliche IP-Adresse an, die dem Gateway für das virtuelle TestLab-Netzwerk zugewiesen wird.

    $gwpip=New-AzureRmPublicIpAddress -Name TestLab_pip -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic

Erstellen Sie im nächsten Schritt Ihr Gateway.

    $vnet=Get-AzureRmVirtualNetwork -Name TestLab -ResourceGroupName $rgName
    $subnet=Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
    $gwipconfig=New-AzureRmVirtualNetworkGatewayIpConfig -Name TestLab_GWConfig -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id 
    New-AzureRmVirtualNetworkGateway -Name TestLab_GW -ResourceGroupName $rgName -Location $locName -IpConfigurations $gwipconfig -GatewayType Vpn -VpnType RouteBased

Bedenken Sie, dass die Erstellung neuer Gateways 20 Minuten oder länger dauern kann.

Stellen Sie im Azure-Portal auf Ihrem lokalen Computer eine Verbindung mit DC1 her. Verwenden Sie dafür die Anmeldeinformationen für CORP\\User1. Führen Sie die folgenden Befehle an einer Windows PowerShell-Eingabeaufforderung auf DC1 aus, um die CORP-Domäne so zu konfigurieren, dass Computer und Benutzer ihren lokalen Domänencontroller zur Authentifizierung verwenden.

    New-ADReplicationSite -Name "TestLab" 
    New-ADReplicationSite -Name "TestVNET"
    New-ADReplicationSubnet -Name "10.0.0.0/8" -Site "TestLab"
    New-ADReplicationSubnet -Name "192.168.0.0/16" -Site "TestVNET"

Die aktuelle Konfiguration sieht folgendermaßen aus.

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-sim/virtual-machines-windows-ps-hybrid-cloud-test-env-sim-ph1.png)

## Phase 2: Erstellen des virtuellen Netzwerks "TestVNET"
Zunächst erstellen Sie das virtuelle Netzwerk „TestVNET“ und schützen es mit einer Netzwerksicherheitsgruppe.

    $rgName="<name of the resource group that you used for your TestLab virtual network>"
    $locName="<Azure location name where you placed the TestLab virtual network, such as West US>"
    $locShortName="<Azure location name from $locName in all lowercase letters with spaces removed. Example:  westus>"
    $testSubnet=New-AzureRMVirtualNetworkSubnetConfig -Name "TestSubnet" -AddressPrefix 192.168.0.0/24
    $gatewaySubnet=New-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix 192.168.255.248/29
    New-AzureRMVirtualNetwork -Name "TestVNET" -ResourceGroupName $rgName -Location $locName -AddressPrefix 192.168.0.0/16 -Subnet $testSubnet,$gatewaySubnet –DNSServer 10.0.0.4
    $rule1=New-AzureRMNetworkSecurityRuleConfig -Name "RDPTraffic" -Description "Allow RDP to all VMs on the subnet" -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389
    New-AzureRMNetworkSecurityGroup -Name "TestSubnet" -ResourceGroupName $rgName -Location $locShortName -SecurityRules $rule1
    $vnet=Get-AzureRMVirtualNetwork -ResourceGroupName $rgName -Name TestVNET
    $nsg=Get-AzureRMNetworkSecurityGroup -Name "TestSubnet" -ResourceGroupName $rgName
    Set-AzureRMVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "TestSubnet" -AddressPrefix 192.168.0.0/24 -NetworkSecurityGroup $nsg

Als Nächstes fordern Sie eine öffentliche IP-Adresse an, die dem Gateway für das virtuelle Netzwerk „TestVNET“ zugeordnet wird, und erstellen Ihr Gateway.

    $gwpip=New-AzureRmPublicIpAddress -Name TestVNET_pip -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
    $vnet=Get-AzureRmVirtualNetwork -Name TestVNET -ResourceGroupName $rgName
    $subnet=Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
    $gwipconfig=New-AzureRmVirtualNetworkGatewayIpConfig -Name "TestVNET_GWConfig" -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id
    New-AzureRmVirtualNetworkGateway -Name "TestVNET_GW" -ResourceGroupName $rgName -Location $locName -IpConfigurations $gwipconfig -GatewayType Vpn -VpnType RouteBased

Die aktuelle Konfiguration sieht folgendermaßen aus.

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-sim/virtual-machines-windows-ps-hybrid-cloud-test-env-sim-ph2.png)

## Phase 3: Erstellen der VNet-zu-VNet-Verbindung
Besorgen Sie sich zunächst einen zufällig generierten vorinstallierten Schlüssel mit 32 Zeichen und starker Kryptografie vom Netzwerk- oder Systemadministrator. Verwenden Sie zum Abrufen eines vorinstallierten Schlüssels alternativ die Informationen im Artikel [Create a random string for an IPsec preshared key](http://social.technet.microsoft.com/wiki/contents/articles/32330.create-a-random-string-for-an-ipsec-preshared-key.aspx) (Erstellen einer zufälligen Zeichenfolge für einen vorinstallierten IPsec-Schlüssel).

Verwenden Sie im nächsten Schritt dies folgenden Befehle, um die VNet-zu-VNet-VPN-Verbindung zu erstellen. Dies kann einige Zeit in Anspruch nehmen.

    $sharedKey="<pre-shared key value>"
    $gwTestLab=Get-AzureRmVirtualNetworkGateway -Name TestLab_GW -ResourceGroupName $rgName
    $gwTestVNET=Get-AzureRmVirtualNetworkGateway -Name TestVNET_GW -ResourceGroupName $rgName
    New-AzureRmVirtualNetworkGatewayConnection -Name TestLab_to_TestVNET -ResourceGroupName $rgName -VirtualNetworkGateway1 $gwTestLab -VirtualNetworkGateway2 $gwTestVNET -Location $locName -ConnectionType Vnet2Vnet -SharedKey $sharedKey
    New-AzureRmVirtualNetworkGatewayConnection -Name TestVNET_to_TestLab -ResourceGroupName $rgName -VirtualNetworkGateway1 $gwTestVNET -VirtualNetworkGateway2 $gwTestLab -Location $locName -ConnectionType Vnet2Vnet -SharedKey $sharedKey

Nach einigen Minuten sollte die Verbindung hergestellt sein.

Die aktuelle Konfiguration sieht folgendermaßen aus.

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-sim/virtual-machines-windows-ps-hybrid-cloud-test-env-sim-ph3.png)

## Phase 4: Konfigurieren von DC2
Erstellen Sie zunächst einen virtuellen Computer für DC2. Führen Sie in der Eingabeaufforderung von Azure PowerShell auf dem lokalen Computer die folgenden Befehle aus.

    $rgName="<your resource group name>"
    $locName="<your Azure location, such as West US>"
    $saName="<the storage account name for the base configuration>"
    $vnet=Get-AzureRMVirtualNetwork -Name TestVNET -ResourceGroupName $rgName
    $pip=New-AzureRMPublicIpAddress -Name DC2-NIC -ResourceGroupName $rgName -Location $locName -AllocationMethod Dynamic
    $nic=New-AzureRMNetworkInterface -Name DC2-NIC -ResourceGroupName $rgName -Location $locName -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -PrivateIpAddress 192.168.0.4
    $vm=New-AzureRMVMConfig -VMName DC2 -VMSize Standard_A1
    $storageAcc=Get-AzureRMStorageAccount -ResourceGroupName $rgName -Name $saName
    $vhdURI=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/DC2-TestVNET-ADDSDisk.vhd"
    Add-AzureRMVMDataDisk -VM $vm -Name ADDS-Data -DiskSizeInGB 20 -VhdUri $vhdURI  -CreateOption empty
    $cred=Get-Credential -Message "Type the name and password of the local administrator account for DC2."
    $vm=Set-AzureRMVMOperatingSystem -VM $vm -Windows -ComputerName DC2 -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
    $vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
    $vm=Add-AzureRMVMNetworkInterface -VM $vm -Id $nic.Id
    $osDiskUri=$storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/DC2-TestLab-OSDisk.vhd"
    $vm=Set-AzureRMVMOSDisk -VM $vm -Name DC2-TestVNET-OSDisk -VhdUri $osDiskUri -CreateOption fromImage
    New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm

Stellen Sie anschließend über das Azure-Portal eine Verbindung mit dem neuen virtuellen Computer DC2 her.

Konfigurieren Sie anschließend eine Windows-Firewall-Regel, um Datenverkehr zum Testen der allgemeinen Konnektivität zuzulassen. Führen Sie in der Windows PowerShell-Eingabeaufforderung von DC2 die folgenden Befehle auf Administratorebene aus.

    Set-NetFirewallRule -DisplayName "File and Printer Sharing (Echo Request - ICMPv4-In)" -enabled True
    ping dc1.corp.contoso.com

Der Ping-Befehl sollte zu vier erfolgreichen Antworten von der IP-Adresse 10.0.0.4 führen. Dadurch wird der Datenverkehr über die VNet-zu-VNet-Verbindung getestet.

Fügen Sie anschließend den zusätzlichen Datenträger auf DC2 als neues Volume mit dem Laufwerkbuchstaben „F:“ hinzu.

1. Klicken Sie im linken Bereich des Server-Managers auf **Datei- und Speicherdienste** und anschließend auf **Datenträger**.
2. Klicken Sie im Inhaltsbereich in der Gruppe **Datenträger** auf **Datenträger 2** (wobei **Partition** die Einstellung **Unbekannt** aufweisen muss).
3. Klicken Sie auf **Aufgaben** und anschließend auf **Neues Volume**.
4. Klicken Sie auf der Seite „Voraussetzungen“ des Assistenten für neue Volumes auf **Weiter**.
5. Klicken Sie auf der Seite „Server und Datenträger auswählen“ auf **Datenträger 2** und anschließend auf **Weiter**. Wenn Sie dazu aufgefordert werden, klicken Sie auf **OK**.
6. Klicken Sie auf der Seite „Geben Sie die Größe des Volumes an“ auf **Weiter**.
7. Klicken Sie auf der Seite „Einem Laufwerkbuchstaben oder Ordner zuweisen“ auf **Weiter**.
8. Klicken Sie auf der Seite „Dateisystemeinstellungen auswählen“ auf **Weiter**.
9. Klicken Sie auf der Seite „Auswahl bestätigen“ auf **Erstellen**.
10. Wenn Sie fertig sind, klicken Sie auf **Schließen**.

Konfigurieren Sie als Nächstes DC2 als replizierten Domänencontroller für die corp.contoso.com-Domäne. Führen Sie in der Windows PowerShell-Eingabeaufforderung von DC2 die folgenden Befehle aus.

    Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
    Install-ADDSDomainController -Credential (Get-Credential CORP\User1) -DomainName "corp.contoso.com" -InstallDns:$true -DatabasePath "F:\NTDS" -LogPath "F:\Logs" -SysvolPath "F:\SYSVOL"

Beachten Sie, dass Sie aufgefordert werden, sowohl das Kennwort für CORP\\User1 als auch ein Kennwort für den Verzeichnisdienste-Wiederherstellungsmodus anzugeben und DC2 neu zu starten.

Da das virtuelle Netzwerk TestVNET nun über einen eigenen DNS-Server (DC2) verfügt, müssen Sie das virtuelle Netzwerk TestVNET zur Verwendung dieses DNS-Servers konfigurieren.

1. Klicken Sie im linken Bereich des Azure-Portals auf das Symbol für virtuelle Netzwerke, und klicken Sie dann auf **TestVNET**.
2. Klicken Sie auf der Registerkarte **Einstellungen** auf **DNS-Server**.
3. Geben Sie unter **Primärer DNS-Server** die Adresse **192.168.0.4** ein, und ersetzen Sie dadurch die Adresse 10.0.0.4.
4. Klicken Sie auf **Speichern**.

Die aktuelle Konfiguration sieht folgendermaßen aus.

![](./media/virtual-machines-windows-ps-hybrid-cloud-test-env-sim/virtual-machines-windows-ps-hybrid-cloud-test-env-sim-ph4.png)

Die simulierte Hybrid Cloud-Umgebung kann jetzt zum Testen verwendet werden.

## Nächster Schritt
* Richten Sie in dieser Umgebung eine [webbasierte Branchenanwendung](virtual-machines-windows-ps-hybrid-cloud-test-env-lob.md) ein.

<!---HONumber=AcomDC_0810_2016-->