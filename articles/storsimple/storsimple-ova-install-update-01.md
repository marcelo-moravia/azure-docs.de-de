---
title: Installieren von Updates auf einem StorSimple Virtual Array | Microsoft Docs
description: Hier wird beschrieben, wie Sie die Webbenutzeroberfläche des StorSimple Virtual Arrays verwenden, um Updates und Hotfixes über das Portal anzuwenden.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''

ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 09/07/2016
ms.author: alkohli

---
# Installieren von Updates auf einem StorSimple Virtual Array
## Übersicht
Dieser Artikel beschreibt die erforderlichen Schritte zum Installieren von Updates auf Ihrem StorSimple Virtual Array über die lokale Webbenutzeroberfläche oder das klassische Azure-Portal. Sie müssen Softwareupdates oder -hotfixes installieren, um Ihr StorSimple Virtual Array auf dem neuesten Stand zu halten.

Beachten Sie, dass das Gerät bei der Installation des Updates oder Hotfixes neu gestartet wird. Da es sich bei einem StorSimple Virtual Array um ein Gerät mit einem Einzelknoten handelt, werden alle aktuellen E/A-Vorgänge unterbrochen, was zu einer gewissen Ausfallzeit führt.

Außerdem wird empfohlen, die Volumes oder Freigaben vor dem Anwenden eines Updates auf dem Host und dann auf dem Gerät offline zu nehmen. Dadurch wird das Risiko einer Datenbeschädigung minimiert.

> [!IMPORTANT]
> Wenn Sie Update 0.1 oder GA-Softwareversionen ausführen, müssen Sie die Hotfixmethode über die lokale Webbenutzeroberfläche verwenden, um Update 0.3 zu installieren. Wenn Sie Update 0.2 ausführen, empfehlen wir, die Software- und Treiberupdates über das klassische Azure-Portal zu installieren.
> 
> 

## Verwenden der lokalen Webbenutzeroberfläche
Die Verwendung der lokalen Webbenutzeroberfläche umfasst zwei Schritte:

* Herunterladen des Updates oder Hotfixes
* Installieren des Updates oder Hotfixes

### Herunterladen des Updates oder Hotfixes
Führen Sie die folgenden Schritte aus, um das Softwareupdate aus dem Microsoft Update-Katalog herunterzuladen.

#### So laden sie das Update oder den Hotfix herunter
1. Starten Sie Internet Explorer, und navigieren Sie zu [http://catalog.update.microsoft.com](http://catalog.update.microsoft.com).
2. Falls Sie den Microsoft Update-Katalog zum ersten Mal auf diesem Computer verwenden, klicken Sie auf **Installieren**, wenn Sie zum Installieren des Add-Ons für den Microsoft Update-Katalog aufgefordert werden.
3. Geben Sie im Suchfeld des Microsoft Update-Katalogs die KB-Nummer (Knowledge Base) des Hotfixes ein, den Sie herunterladen möchten. Geben Sie **3182061** für Update 0.3 ein, und klicken Sie anschließend auf **Suchen**.
   
    Die Hotfixliste wird angezeigt, z.B. **StorSimple Virtual Array Update 0.3**.
   
    ![Katalog durchsuchen](./media/storsimple-ova-install-update-01/download1.png)
4. Klicken Sie auf **Hinzufügen**. Das Update wird dem Warenkorb hinzugefügt.
5. Klicken Sie auf **Warenkorb anzeigen**.
6. Klicken Sie auf **Download**. Geben Sie einen lokalen Speicherort an, oder klicken Sie auf **Durchsuchen**, und navigieren Sie zum gewünschten lokalen Speicherort für den Download. Die Updates werden heruntergeladen und am angegebenen Speicherort in einem nach dem Update benannten Unterordner gespeichert. Der Ordner kann auch in eine Netzwerkfreigabe kopiert werden, auf die vom Gerät aus zugegriffen werden kann.
7. Öffnen Sie den kopierten Ordner. Es sollte eine Datei des Typs „Eigenständiges Microsoft Update-Paket“ (`WindowsTH-KB3011067-x64`) angezeigt werden. Diese Datei wird verwendet, um das Update oder den Hotfix zu installieren.

### Installieren des Updates oder Hotfixes
Stellen Sie vor der Update- oder Hotfixinstallation sicher, dass das Update oder der Hotfix entweder lokal auf Ihren Host heruntergeladen wurde bzw. dass auf das Update oder den Hotfix über eine Netzwerkfreigabe zugegriffen werden kann.

Verwenden Sie diese Methode zum Installieren von Updates auf einem Gerät, auf dem GA- oder Update 0.1-Softwareversionen ausgeführt werden. Dieses Verfahren dauert weniger als zwei Minuten. Führen Sie die folgenden Schritte aus, um Updates oder Hotfixes zu installieren.

#### So installieren Sie das Update oder den Hotfix
1. Wechseln Sie auf der lokalen Webbenutzeroberfläche zu **Wartung** > **Softwareupdate**.
   
    ![Gerät aktualisieren](./media/storsimple-ova-install-update-01/update1m.png)
2. Geben Sie in **Updatedateipfad** den Dateinamen für das Update oder den Hotfix ein. Sie können auch zur Installationsdatei des Updates oder Hotfixes navigieren, sofern sie auf einer Netzwerkfreigabe abgelegt wurde. Klicken Sie auf **Übernehmen**.
   
    ![Gerät aktualisieren](./media/storsimple-ova-install-update-01/update2m.png)
3. Es wird eine Warnung angezeigt. Da es sich hierbei um ein Gerät mit einem Einzelknoten handelt, wird das Gerät neu gestartet, und es kommt zu Ausfallzeit. Klicken Sie auf das Häkchensymbol.
   
   ![Gerät aktualisieren](./media/storsimple-ova-install-update-01/update3m.png)
4. Das Update wird gestartet. Nachdem das Gerät erfolgreich aktualisiert wurde, wird es neu gestartet. Auf die lokale Benutzeroberfläche kann währenddessen nicht zugegriffen werden.
   
    ![Gerät aktualisieren](./media/storsimple-ova-install-update-01/update5m.png)
5. Nach dem Neustart wird die Seite **Anmelden** angezeigt. Um sicherzustellen, dass die Gerätesoftware aktualisiert wurde, wechseln Sie auf der lokalen Webbenutzeroberfläche zu **Wartung** > **Softwareupdate**. Die angezeigte Softwareversion sollte **10.0.0.0.0.10288.0** für Update 0.3 lauten.
   
   > [!NOTE]
   > Die Softwareversionen werden in der lokalen Webbenutzeroberfläche und im klassischen Azure-Portal etwas unterschiedlich angegeben. Beispiel: Die lokale Webbenutzeroberfläche gibt **10.0.0.0.0.10288** an, und das klassische Azure-Portal **10.0.10288.0**, obwohl die Version die gleiche ist.
   > 
   > 
   
    ![Gerät aktualisieren](./media/storsimple-ova-install-update-01/update6m.png)

## Verwenden des klassischen Azure-Portals
Wir empfehlen Ihnen bei Ausführung von Update 0.2, Updates über das klassische Azure-Portal zu installieren. Beim Verfahren über das Portal muss der Benutzer die Updates suchen, herunterladen und anschließend installieren. Für dieses Verfahren werden ca. 7 Minuten benötigt. Führen Sie die folgenden Schritte aus, um Updates oder Hotfixes zu installieren.

[!INCLUDE [storsimple-ova-install-update-via-portal](../../includes/storsimple-ova-install-update-via-portal.md)]

Wechseln Sie nach Abschluss der Installation (wenn der Auftragsstatus bei 100 % ist) zu **Geräte > Wartung > Softwareupdates**. Die angezeigte Softwareversion sollte 10.0.10288.0 lauten.

![Gerät aktualisieren](./media/storsimple-ova-install-update-01/azupdate12m.png)

## Nächste Schritte
Erfahren Sie, wie Sie [StorSimple Virtual Array verwalten](storsimple-ova-web-ui-admin.md).

<!---HONumber=AcomDC_0914_2016-->