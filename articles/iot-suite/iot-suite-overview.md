---
title: "Übersicht über Microsoft Azure IoT Suite | Microsoft Docs"
description: "Hier erhalten Sie einen Überblick über die von Azure IoT Suite bereitgestellten vorkonfigurierten IoT-Lösungen zum Erfassen, Analysieren und Speichern von Daten, zum Bereitstellen von Visualisierungen und Integrieren in andere Systeme."
services: 
suite: iot-suite
documentationcenter: 
author: dominicbetts
manager: timlt
editor: 
ms.assetid: 2d38d08a-4133-4e5c-8b28-f93cadb5df05
ms.service: iot-suite
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/09/2016
ms.author: dobett
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 7b0df1fc79e82a99ea65296a4e09bdb48fde9451


---
# <a name="what-is-azure-iot-suite"></a>Was ist Azure IoT Suite?
Die Azure IoT-Dienste (Internet of Things, Internet der Dinge) bieten zahlreiche Funktionen. Diese Dienste auf Unternehmensniveau ermöglichen Folgendes:

* Sammeln von Daten von Geräten
* Analysieren von Datenströmen während des Betriebs
* Speichern und Abfragen von großen Datasets
* Anzeigen von Echtzeit- und Verlaufsdaten
* Integrieren mit Back-Office-Systemen

Zur Bereitstellung dieser Funktionen kombiniert die Azure IoT Suite mehrere Azure-Dienste mit benutzerdefinierten Erweiterungen als *vorkonfigurierte Lösungen*. Diese vorkonfigurierten Lösungen sind Basisimplementierungen von häufig verwendeten IoT-Lösungsmustern, mit denen Sie die Bereitstellung Ihrer IoT-Lösungen beschleunigen können. Mithilfe der [IoT-Software Development Kits][lnk-sdks] können Sie diese Lösungen Ihren Anforderungen anpassen und ggf. erweitern. Wenn Sie neue IoT-Lösungen entwickeln, können Sie diese Lösungen auch als Beispiele oder Vorlagen verwenden.

Das folgende Video bietet eine Einführung in Azure IoT Suite:

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON309/player]
> 
> 

## <a name="azure-iot-services-in-azure-iot-suite"></a>Azure IoT-Dienste in Azure IoT Suite
Die vorkonfigurierten Lösungen verwenden in der Regel die folgenden Dienste:

* Kern der Azure IoT Suite ist der [Azure IoT Hub][lnk-iot-hub]-Dienst. Dieser Dienst stellt die D2C- und C2D-Messagingfunktionen (Gerät zu Cloud und Cloud zu Gerät) und fungiert als Gateway für die Cloud und andere wichtige Dienste der IoT-Suite. Der Dienst ermöglicht das Empfangen von Nachrichten von Ihren verwendeten Geräten und das Senden von Befehlen an diese Geräte.
* Die Datenanalyse während des Betriebs wird von [Azure Stream Analytics][lnk-asa] bereitgestellt. Die IoT Suite nutzt diesen Dienst für die Verarbeitung eingehender Telemetriedaten, für die Aggregation und für das Erkennen von Ereignissen. Die vorkonfigurierten Lösungen verwenden Stream Analytics auch für die Verarbeitung informativer Meldungen, die Daten wie Metadaten oder Antworten auf Befehle von Geräten enthalten. Die Lösungen verwenden Stream Analytics zum Verarbeiten von Nachrichten von Ihren Geräten und zum Übermitteln dieser Nachrichten an andere Dienste.
* [Azure Storage][lnk-azure-storage] und [Azure DocumentDB][lnk-document-db] bieten Datenspeicherfunktionen. Die vorkonfigurierten Lösungen verwenden den Blob-Speicher, um Telemetriedaten zu speichern und für die Analyse verfügbar zu machen. Die Lösungen verwenden DocumentDB zum Speichern von Metadaten zum Gerät und zum Aktivieren der Verwaltungsfunktionen der Lösungen für Geräte.
* [Azure-Web-Apps][lnk-web-apps] und [Microsoft Power BI][lnk-power-bi] bieten Datenvisualisierungsfunktionen. Die Flexibilität von Power BI ermöglicht Kunden das schnelle Erstellen eigener interaktiver Dashboards, die Daten der IoT Suite verwenden.

Eine Übersicht über die Architektur einer typischen IoT-Lösung finden Sie unter [Microsoft Azure und das Internet der Dinge (IoT)][iot-suite-what-is-azure-iot].

## <a name="preconfigured-solutions"></a>Vorkonfigurierte Lösungen
Die IoT Suite enthält vorkonfigurierte Lösungen, die einen schneller Einstieg und Einblick in die mit der Azure IoT Suite möglichen IoT-Szenarien ermöglichen, wie etwa *Remoteüberwachung* und *Vorhersagbarer Wartungsbedarf*. Sie können die Lösungen für Ihr Azure-Abonnement bereitstellen und dann ein vollständiges End-to-End-IoT-Szenario ausführen.

## <a name="next-steps"></a>Nächste Schritte
Sie haben einen Überblick über die Möglichkeiten der IoT-Suite erlangt und kennen ihre wichtigsten Komponenten. Unter [Was sind vorkonfigurierte Azure IoT-Suite-Lösungen?][lnk-what-are-preconfig] erfahren Sie mehr über die vorkonfigurierten Lösungen in IoT Suite.

[lnk-sdks]: https://azure.microsoft.com/documentation/articles/iot-hub-sdks-summary/
[lnk-iot-hub]: https://azure.microsoft.com/documentation/services/iot-hub/
[lnk-asa]: https://azure.microsoft.com/documentation/services/stream-analytics/
[lnk-azure-storage]: https://azure.microsoft.com/documentation/services/storage/
[lnk-document-db]: https://azure.microsoft.com/documentation/services/documentdb/
[lnk-power-bi]: https://powerbi.microsoft.com/
[lnk-web-apps]: https://azure.microsoft.com/documentation/services/app-service/web/
[iot-suite-what-is-azure-iot]: iot-suite-what-is-azure-iot.md
[lnk-what-are-preconfig]: iot-suite-what-are-preconfigured-solutions.md



<!--HONumber=Nov16_HO2-->


