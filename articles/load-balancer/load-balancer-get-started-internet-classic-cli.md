---
title: Erste Schritte zum Erstellen eines Lastenausgleichs mit Internetzugriff im klassischen Bereitstellungsmodell mithilfe der Azure-Befehlszeilenschnittstelle | Microsoft Docs
description: Erfahren Sie, wie Sie mithilfe der Azure-Befehlszeilenschnittstelle einen Load Balancer mit Internetzugriff in einem klassischen Bereitstellungsmodell erstellen.
services: load-balancer
documentationcenter: na
author: sdwheeler
manager: carmonm
editor: 
tags: azure-service-management
ms.assetid: e433a824-4a8a-44d2-8765-a74f52d4e584
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/09/2016
ms.author: sewhee
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: e8a346c1b2d430eceb4aa1b8bc94fbbe89394556


---
# <a name="get-started-creating-an-internet-facing-load-balancer-classic-in-the-azure-cli"></a>Erste Schritte zum Erstellen eines Load Balancers mit Internetzugriff (klassisch) mithilfe der Azure-Befehlszeilenschnittstelle
[!INCLUDE [load-balancer-get-started-internet-classic-selectors-include.md](../../includes/load-balancer-get-started-internet-classic-selectors-include.md)]

[!INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

Dieser Artikel gilt für das klassische Bereitstellungsmodell. Informationen zum Erstellen eines Load Balancers mit Internetzugriff unter Verwendung des Azure-Ressourcen-Managers finden Sie [hier](load-balancer-get-started-internet-arm-ps.md).

[!INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

## <a name="step-by-step-creating-an-internet-facing-load-balancer-using-cli"></a>Schritte zum Erstellen eines Load Balancers mit Internetzugriff mithilfe der Befehlszeilenschnittstelle
In diesem Handbuch erfahren Sie, wie Sie auf der Grundlage des oben beschriebenen Szenarios einen Internet-Load Balancer erstellen.

1. Wenn Sie die Azure-Befehlszeilenschnittstelle noch nie verwendet haben, ziehen Sie [Installieren und Konfigurieren der Azure-Befehlszeilenschnittstelle](../xplat-cli-install.md) zurate, und folgen Sie den Anweisungen bis zu dem Punkt, an dem Sie Ihr Azure-Konto und Ihr Abonnement auswählen.
2. Führen Sie den Befehl **azure config mode** aus, um in den klassischen Modus zu wechseln, wie unten dargestellt.
   
        azure config mode asm
   
    Erwartete Ausgabe:
   
        info:    New mode is asm

## <a name="create-endpoint-and-load-balancer-set"></a>Erstellen eines Endpunkts und einer Load Balancer-Gruppe
Das Szenario setzt voraus, dass die virtuellen Computer „web1“ und „web2“ erstellt wurden.
In diesem Leitfaden wird eine Load Balancer-Gruppe erstellt, die Port 80 als öffentlichen Port und Port 80 als lokalen Port verwendet. An Port 80 wird zudem ein Testport konfiguriert und als Load Balancer-Gruppe „lbset“ bezeichnet.

### <a name="step-1"></a>Schritt 1
Erstellen des ersten Endpunkts und einer Load Balancer-Gruppe für den virtuellen Computer „web1“ mithilfe von `azure network vm endpoint create` .

    azure vm endpoint create web1 80 -k 80 -o tcp -t 80 -b lbset

Verwendete Parameter:

**-k**: Port des lokalen virtuellen Computers<br>
**-o**: Protokoll<BR>
**-t**: Testport<BR>
**-b**: Name des Load Balancers<BR>

## <a name="step-2"></a>Schritt 2
Hinzufügen eines zweiten virtuellen Computers namens „web2“ zur Load Balancer-Gruppe

    azure vm endpoint create web2 80 -k 80 -o tcp -t 80 -b lbset

## <a name="step-3"></a>Schritt 3
Überprüfen der Load Balancer-Konfiguration mithilfe von `azure vm show` .

    azure vm show web1

Ausgabe:

    data:    DNSName "contoso.cloudapp.net"
    data:    Location "East US"
    data:    VMName "web1"
    data:    IPAddress "10.0.0.5"
    data:    InstanceStatus "ReadyRole"
    data:    InstanceSize "Standard_D1"
    data:    Image "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-2015
    6-en.us-127GB.vhd"
    data:    OSDisk hostCaching "ReadWrite"
    data:    OSDisk name "joaoma-1-web1-0-201509251804250879"
    data:    OSDisk mediaLink "https://XXXXXXXXXXXXXXX.blob.core.windows.
    /vhds/joaomatest-web1-2015-09-25.vhd"
    data:    OSDisk sourceImageName "a699494373c04fc0bc8f2bb1389d6106__Windows-Se
    r-2012-R2-20150916-en.us-127GB.vhd"
    data:    OSDisk operatingSystem "Windows"
    data:    OSDisk iOType "Standard"
    data:    ReservedIPName ""
    data:    VirtualIPAddresses 0 address "XXXXXXXXXXXXXXXX"
    data:    VirtualIPAddresses 0 name "XXXXXXXXXXXXXXXXXXXX"
    data:    VirtualIPAddresses 0 isDnsProgrammed true
    data:    Network Endpoints 0 loadBalancedEndpointSetName "lbset"
    data:    Network Endpoints 0 localPort 80
    data:    Network Endpoints 0 name "tcp-80-80"
    data:    Network Endpoints 0 port 80
    data:    Network Endpoints 0 loadBalancerProbe port 80
    data:    Network Endpoints 0 loadBalancerProbe protocol "tcp"
    data:    Network Endpoints 0 loadBalancerProbe intervalInSeconds 15
    data:    Network Endpoints 0 loadBalancerProbe timeoutInSeconds 31
    data:    Network Endpoints 0 protocol "tcp"
    data:    Network Endpoints 0 virtualIPAddress "XXXXXXXXXXXX"
    data:    Network Endpoints 0 enableDirectServerReturn false
    data:    Network Endpoints 1 localPort 5986
    data:    Network Endpoints 1 name "PowerShell"
    data:    Network Endpoints 1 port 5986
    data:    Network Endpoints 1 protocol "tcp"
    data:    Network Endpoints 1 virtualIPAddress "XXXXXXXXXXXX"
    data:    Network Endpoints 1 enableDirectServerReturn false
    data:    Network Endpoints 2 localPort 3389
    data:    Network Endpoints 2 name "Remote Desktop"
    data:    Network Endpoints 2 port 58081
    info:    vm show command OK

## <a name="create-a-remote-desktop-endpoint-for-a-virtual-machine"></a>Erstellen eines Remotedesktopendpunkts für einen virtuellen Computer
Sie können mithilfe von `azure vm endpoint create`einen Remotedesktop-Endpunkt erstellen, um für einen bestimmten virtuellen Computer Netzwerkdatenverkehr von einem öffentlichen Port an einen lokalen Port weiterzuleiten.

    azure vm endpoint create web1 54580 -k 3389


## <a name="remove-virtual-machine-from-load-balancer"></a>Entfernen eines virtuellen Computers aus einem Load Balancer
Sie müssen den der Load Balancer-Gruppe zugeordneten Endpunkt vom virtuellen Computer löschen. Sobald der Endpunkt entfernt wurde, gehört der virtuelle Computer nicht mehr zur Load Balancer-Gruppe.

 Anhand des oben genannten Beispiels können Sie mithilfe des Befehls `azure vm endpoint delete` den für den virtuellen Computer „web1“ erstellten Endpunkt aus dem Load Balancer „lbset“ entfernen.

    azure vm endpoint delete web1 tcp-80-80


> [!NOTE]
> Mit dem Befehl `azure vm endpoint --help` können Sie weitere Optionen zum Verwalten von Endpunkten ausprobieren.
> 
> 

## <a name="next-steps"></a>Nächste Schritte
[Erste Schritte zum Konfigurieren des internen Lastenausgleichs](load-balancer-get-started-ilb-arm-ps.md)

[Konfigurieren eines Lastenausgleichs-Verteilungsmodus](load-balancer-distribution-mode.md)

[Konfigurieren von TCP-Leerlauftimeout-Einstellungen für den Lastenausgleich](load-balancer-tcp-idle-timeout.md)




<!--HONumber=Nov16_HO2-->


