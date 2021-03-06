---
title: "Azure AD-Domänendienste: Erstellen oder Auswählen eines virtuellen Netzwerks | Microsoft Docs"
description: "Erste Schritte mit Azure Active Directory-Domänendiensten"
services: active-directory-ds
documentationcenter: 
author: mahesh-unnikrishnan
manager: stevenpo
editor: curtand
ms.assetid: 13ab1608-e3d8-40de-9f7b-9b5b42199af4
ms.service: active-directory-ds
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 10/03/2016
ms.author: maheshu
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 0617dbff2ba281091ba5775e7969edfead4d80c8


---
# <a name="create-or-select-a-virtual-network-for-azure-ad-domain-services"></a>Erstellen oder Auswählen eines virtuellen Netzwerks für Azure AD-Domänendienste
## <a name="guidelines-to-select-an-azure-virtual-network"></a>Richtlinien zur Auswahl eines virtuellen Azure-Netzwerks
> [!NOTE]
> **Voraussetzungen**: Lesen Sie [Networking considerations for Azure AD Domain Services](active-directory-ds-networking.md)(Netzwerkaspekte für Azure AD-Domänendienste).
> 
> 

## <a name="task-2-create-an-azure-virtual-network"></a>Aufgabe 2: Erstellen eines virtuellen Azure-Netzwerks
Die nächste Konfigurationsaufgabe besteht in der Erstellung eines virtuellen Azure-Netzwerks und eines dazugehörigen Subnetzes. Azure AD Domain Services wird dann in diesem Subnetz Ihres virtuellen Netzwerks aktiviert. Wenn Sie bereits über ein vorhandenes virtuelles Netzwerk verfügen, das Sie verwenden möchten, können Sie diesen Schritt überspringen.

> [!NOTE]
> Stellen Sie sicher, dass das virtuelle Azure-Netzwerk, das Sie für Azure Active Directory-Domänendienste erstellen oder verwenden möchten, einer Azure-Region angehört, die von Azure Active Directory-Domänendiensten unterstützt wird. Informationen zu den Azure-Regionen, in denen Azure AD-Domänendienste verfügbar sind, finden Sie unter [Azure-Dienste nach Region](https://azure.microsoft.com/regions/#services/) .
> 
> 

Notieren Sie sich den Namen des virtuellen Netzwerks, damit Sie das richtige virtuelle Netzwerk auswählen, wenn Sie Azure Active Directory-Domänendienste in einem nachfolgenden Konfigurationsschritt aktivieren.

Führen Sie die folgenden Konfigurationsschritte aus, um ein virtuelles Azure-Netzwerk zu erstellen, in dem Sie Azure Active Directory-Domänendienste aktivieren möchten.

1. Navigieren Sie zum **klassischen Azure-Portal** ([https://manage.windowsazure.com](https://manage.windowsazure.com)).
2. Wählen Sie den Knoten **Netzwerke** auf der linken Seite aus.
   
    ![Knoten „Netzwerke“](./media/active-directory-domain-services-getting-started/networks-node.png)
3. Klicken Sie im Aufgabenbereich unten auf der Seite auf **NEU** .
   
    ![Knoten „Virtuelle Netzwerke“](./media/active-directory-domain-services-getting-started/virtual-networks.png)
4. Wählen Sie **Virtuelles Netzwerk** im Knoten **Netzwerkdienste**.
5. Klicken Sie auf **Schnellerfassung** , um ein virtuelles Netzwerk zu erstellen.
   
    ![Virtuelles Netzwerk – Schnellerfassung](./media/active-directory-domain-services-getting-started/virtual-network-quickcreate.png)
6. Geben Sie einen **Namen** für Ihr virtuelles Netzwerk an. Optional können Sie den **Adressraum** oder die **maximale Anzahl virtueller Computer** für dieses Netzwerk konfigurieren. Sie können die Einstellung **DNS-Server** vorerst auf „Kein“ festgelegt lassen. Sie können diese Einstellung nach der Aktivierung von Azure AD Domain Services aktualisieren.
7. Stellen Sie sicher, dass Sie in der Dropdownliste **Standort** eine unterstützte Azure-Region auswählen. Informationen zu den Azure-Regionen, in denen Azure AD-Domänendienste verfügbar sind, finden Sie unter [Azure-Dienste nach Region](https://azure.microsoft.com/regions/#services/) .
8. Klicken Sie auf die Schaltfläche **Virtuelles Netzwerk erstellen** , um das virtuelle Netzwerk zu erstellen.
   
    ![Erstellen Sie ein virtuelles Netzwerk für Azure Active Directory-Domänendienste.](./media/active-directory-domain-services-getting-started/create-vnet.png)
9. Wählen Sie das erstellte virtuelle Netzwerk aus, und klicken Sie auf die Registerkarte **KONFIGURIEREN**.
   
    ![Erstellen eines Subnetzes](./media/active-directory-domain-services-getting-started/create-vnet-properties.png)
10. Navigieren Sie zum Abschnitt **Adressräume von Virtual Network**. Klicken Sie auf **Subnetz hinzufügen**, und geben Sie ein Subnetz namens **AaddsSubnet** an. Klicken Sie auf **Speichern**, um das Subnetz zu erstellen.
    
    ![Erstellen Sie ein Subnetz für Azure AD Domain Services.](./media/active-directory-domain-services-getting-started/create-vnet-add-subnet.png)

<br>

## <a name="task-3-enable-azure-ad-domain-services"></a>Aufgabe 3: Aktivieren der Azure AD-Domänendienste
Die nächste Konfigurationsaufgabe besteht in der [Aktivierung der Azure AD-Domänendienste](active-directory-ds-getting-started-enableaadds.md).




<!--HONumber=Nov16_HO2-->


