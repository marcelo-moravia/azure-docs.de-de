---
title: Erfassen eines Images eines virtuellen Linux-Computers | Microsoft Docs
description: Erfahren Sie, wie Sie ein Image eines mit dem klassischen Bereitstellungsmodell erstellten virtuellen Azure-Computers unter Linux erfassen können.
services: virtual-machines-linux
documentationcenter: ''
author: iainfoulds
manager: timlt
editor: tysonn
tags: azure-service-management

ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/31/2016
ms.author: iainfou

---
# Erfassen eines klassischen virtuellen Linux-Computers als Image
[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]

Erfahren Sie, wie Sie [diese Schritte mit dem Resource Manager-Modell ausführen](virtual-machines-linux-capture-image.md).

In diesem Artikel wird erläutert, wie Sie einen klassischen virtuellen Azure-Computer, auf dem Linux ausgeführt wird, als Image erfassen, um weitere virtuelle Computer zu erstellen. Dieses Image umfasst den Betriebssystemdatenträger und die an den virtuellen Computer angefügten Datenträger. Da das Image keine Netzwerkkonfiguration enthält, müssen Sie die Konfiguration später vornehmen, wenn Sie die anderen virtuellen Computer auf der Basis dieses Images erstellen.

Azure speichert das Image unter **Images**, zusammen mit allen Images, die Sie hochgeladen haben. Weitere Informationen zu Images finden Sie unter [Informationen zu virtuellen Computern in Azure][Informationen zu virtuellen Computern in Azure].

## Voraussetzungen
Diese Schritte setzen voraus, dass Sie bereits mithilfe des klassischen Bereitstellungsmodells einen virtuellen Azure-Computer erstellt, das Betriebssystem konfiguriert und Datenträger angefügt haben. Falls Sie einen virtuellen Computer erstellen müssen, lesen Sie sich [Erstellen eines benutzerdefinierten virtuellen Linux-Computers][Erstellen eines benutzerdefinierten virtuellen Linux-Computers] durch.

## Erfassen des virtuellen Computers
1. Stellen Sie über einen SSH-Client Ihrer Wahl eine [Verbindung mit dem virtuellen Computer](virtual-machines-linux-mac-create-ssh-keys.md) her.
2. Geben Sie im SSH-Fenster den folgenden Befehl ein. Die Ausgabe von `waagent` kann je nach Version dieses Hilfsprogramms geringfügig abweichen:
   
    `sudo waagent -deprovision+user`
   
    Dieser Befehl versucht, das System zu bereinigen und für eine erneute Bereitstellung vorzubereiten. Bei diesem Vorgang werden die folgenden Aufgaben ausgeführt:
   
   * Entfernen von SSH-Hostschlüsseln (sofern "Provisioning.RegenerateSshHostKeyPair" in der Konfigurationsdatei auf "y" festgelegt ist)
   * Löschen der Namenserverkonfiguration in "/etc/resolv.conf"
   * Entfernen des Kennworts des `root`-Benutzers aus „/etc/shadow“ (sofern „Provisioning.DeleteRootPassword“ in der Konfigurationsdatei auf „y“ festgelegt ist)
   * Entfernen von zwischengespeicherten DHCP-Clientleases
   * Setzt den Hostnamen auf "localhost.localdomain" zurück
   * Löschen des zuletzt bereitgestellten Benutzerkontos (aus "/var/lib/waagent" abgerufen) **und der zugehörigen Daten**
     
     > [!NOTE]
     > Beim Aufheben der Bereitstellung werden Dateien und Daten gelöscht, um das Image zu „generalisieren“. Führen Sie diesen Befehl nur auf einem virtuellen Computer aus, den Sie als neue Imagevorlage erfassen möchten. Dies garantiert nicht, dass alle vertraulichen Informationen aus dem Image gelöscht werden oder dass es für eine erneute Verteilung an Dritte genutzt werden kann.
     > 
     > 
3. Geben Sie **y** ein, um fortzufahren. Sie können den Parameter `-force` hinzufügen, um diesen Bestätigungsschritt zu vermeiden.
4. Geben Sie **Exit** ein, um den SSH-Client zu schließen.
   
   > [!NOTE]
   > Bei den nächsten Schritten wird davon ausgegangen, dass Sie [die Azure-Befehlszeilenschnittstelle auf dem Clientcomputer installiert haben](../xplat-cli-install.md). Die folgenden Schritte können Sie auch im [klassischen Azure-Portal][klassischen Azure-Portal] ausführen.
   > 
   > 
5. Öffnen Sie auf dem Clientcomputer die Azure-CLI, und melden Sie sich bei Ihrem Azure-Abonnement an. Weitere Informationen hierzu finden Sie unter [Herstellen einer Verbindung mit einem Azure-Abonnement von der Azure-Befehlszeilenschnittstelle](../xplat-cli-connect.md).
6. Stellen Sie sicher, dass der Dienstverwaltungsmodus aktiviert ist:
   
    `azure config mode asm`
7. Beenden Sie den virtuellen Computer, dessen Bereitstellung mit den vorherigen Schritten bereits aufgehoben wurde:
   
    `azure vm shutdown <your-virtual-machine-name>`
   
   > [!NOTE]
   > Mit `azure vm list` können Sie alle in Ihrem Abonnement erstellten virtuellen Computer ermitteln.
   > 
   > 
8. Wenn der virtuelle Computer beendet wird, erfassen Sie das Image mit dem folgenden Befehl:
   
    `azure vm capture -t <your-virtual-machine-name> <new-image-name>`
   
    Geben Sie für *new-image-name* den gewünschten Imagenamen ein. Dieser Befehl erstellt ein generalisiertes Betriebssystemimage. Der Unterbefehl `-t` löscht den ursprünglichen virtuellen Computer.
9. Das neue Image ist jetzt in der Liste der Images verfügbar, die zum Konfigurieren neuer virtueller Computer verwendet werden können. Sie können es mit dem folgenden Befehl anzeigen:
   
   `azure vm image list`
   
   Im [klassischen Azure-Portal][klassischen Azure-Portal] wird es in der Liste **IMAGES** angezeigt.
   
   ![Image-Erfassung erfolgreich](./media/virtual-machines-linux-classic-capture-image/VMCapturedImageAvailable.png)

## Nächste Schritte
Das Image kann jetzt zum Erstellen virtueller Computer verwendet werden. Sie können den Azure-CLI-Befehl `azure vm create` verwenden und den Namen des Images angeben, das Sie erstellt haben. Weitere Informationen zu diesem Befehl finden Sie unter [Verwenden der Azure-Befehlszeilenschnittstelle beim klassischen Bereitstellungsmodell](../virtual-machines-command-line-tools.md). Alternativ können Sie einen benutzerdefinierten virtuellen Computer über das [klassische Azure-Portal][klassische Azure-Portal] erstellen, indem Sie die Methode **Aus Katalog** verwenden und das Image auswählen, das Sie erstellt haben. Weitere Informationen finden Sie unter [Erstellen eines benutzerdefinierten virtuellen Computers][Erstellen eines benutzerdefinierten virtuellen Computers].

**Siehe auch**: [Benutzerhandbuch für Azure Linux-Agent](virtual-machines-linux-agent-user-guide.md)

[klassische Azure-Portal]: http://manage.windowsazure.com
[klassischen Azure-Portal]: http://manage.windowsazure.com
[Informationen zu virtuellen Computern in Azure]: virtual-machines-linux-classic-about-images.md
[Erstellen eines benutzerdefinierten virtuellen Computers]: virtual-machines-linux-classic-create-custom.md
[How to Attach a Data Disk to a Virtual Machine]: virtual-machines-windows-classic-attach-disk.md
[Erstellen eines benutzerdefinierten virtuellen Linux-Computers]: virtual-machines-linux-classic-create-custom.md

<!---HONumber=AcomDC_0907_2016-->