---
title: Skalieren von API-Endpunkten | Microsoft Docs
description: Skalieren von Webdienst-Endpunkten in Azure Machine Learning
services: machine-learning
documentationcenter: ''
author: hiteshmadan
manager: padou
editor: ''

ms.service: machine-learning
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 05/25/2016
ms.author: himad

---
# Skalieren von API-Endpunkten
Webdienst-Endpunkte in Azure Machine Learning verfügen über auswählbare Drosselungsstufen, mit denen sie sich an die Nutzungsrate anpassen lassen.

Um die Drosselung des Endpunkts zu steuern, legen Sie mithilfe des Schiebereglers im klassischen Azure-Portal die maximale Anzahl gleichzeitiger Aufrufe auf einen Wert zwischen 20 und 200 fest.

Die synchronen APIs werden in der Regel in Situationen verwendet, in denen eine niedrige Latenz erwünscht ist. Mit Latenz ist hier der Zeitraum gemeint, der von der API benötigt wird, um eine Anforderung abzuschließen, wobei keinerlei Netzwerkverzögerungen berücksichtigt werden. Angenommen, Sie haben eine API mit einer Latenz von 50 ms (Millisekunden). Um die verfügbare Kapazität mit der Drosselungsstufe High (Hoch) und einer Höchstzahl gleichzeitiger Anrufe = 20 voll nutzen zu können, müssen Sie diese API 20 * 1000 / 50 = 400 Mal pro Sekunde aufrufen. Wenn man dieses Beispiel weiter fortspinnt, dann ermöglicht eine Höchstzahl von 200 gleichzeitigen Aufrufen 4000 Aufrufe der API pro Sekunde, sofern die Latenz 50 ms beträgt.

Wenn Sie API stärker auslasten möchten, als sich durch 200 gleichzeitige Aufrufe unterstützen lässt, sollten Sie mehrere Endpunkte im gleichen Webdienst erstellen und die Last beliebig über alle Endpunkte verteilen.

Bedenken Sie, dass sich die Verwendung einer hohen Anzahl gleichzeitiger Aufrufe nachteilig auswirken kann, wenn die API nicht entsprechend ausgelastet wird. Es können sporadische Timeouts und/oder Spitzen in die Latenz auftreten, wenn eine für eine hohe Auslastung konfigurierte API relativ gering ausgelastet wird.

Beachten Sie, dass sich die Anpassung der Drosselungseinstellungen nur auf das Verhalten der synchronen API (RRS) auswirkt. Sie sollten diese Einstellungen anpassen, wenn die synchrone API häufig mit der Meldung "503 Dienst nicht verfügbar" reagiert.

Sie können in der Verwaltungsoberfläche eine benutzerdefinierte Zahl für die Parallelität zum Skalieren des Endpunkts angeben, die über die standardmäßige Parallelität von 20 hinausgeht.

1. Öffnen Sie "manage.windowsazure.com".
2. Navigieren Sie zur Registerkarte "Machine Learning".
3. Klicken Sie auf Ihren Arbeitsbereich.
4. Navigieren Sie zu dem Webdienst mit dem Endpunkt ![Navigieren zum Webdienst](./media/machine-learning-scaling-endpoints/figure-1.png).
5. Klicken Sie auf den Endpunkt anschließend auf die Registerkarte "Configure" (Konfigurieren) ![Navigieren zur Endpunktkonfiguration](./media/machine-learning-scaling-webservice/machlearn-2.png).
6. Ändern Sie den Schieberegler, um den Grad an Parallelität zu erhöhen, und klicken Sie auf "Speichern".

<!---HONumber=AcomDC_0622_2016-->