---
title: RSS | Microsoft Docs
description: Erstellen Sie Logik-Apps mit Azure App Service. Der RSS-Connector ermöglicht Benutzers das Veröffentlichen und Abrufen von Feedelementen. Mit ihm können Benutzer auch Operationen auslösen, wenn ein Element im Feed veröffentlicht wird.
services: logic-apps
documentationcenter: .net,nodejs,java
author: msftman
manager: erikre
editor: ''
tags: connectors

ms.service: logic-apps
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 08/18/2016
ms.author: deonhe

---
# Erste Schritte mit dem RSS-Connector
RSS ist ein beliebtes Webformat zum Veröffentlichen von Inhalten, die häufig aktualisiert werden. Beispiele hierfür wären etwa Blogbeiträge und Schlagzeilen. Viele Inhaltsherausgeber bieten einen RSS-Feed an, der von Benutzern abonniert werden kann. Mit dem RSS-Connector können Sie Feedinformationen abrufen und Abläufe auslösen, wenn in einem RSS-Feed neue Elemente veröffentlicht werden.

> [!NOTE]
> Diese Version des Artikels gilt für die Schemaversion 2015-08-01-preview für Logik-Apps.
> 
> 

Erstellen Sie zu Beginn eine Logik-App, wie unter [Erstellen einer Logik-App](../app-service-logic/app-service-logic-create-a-logic-app.md) beschrieben.

## Trigger und Aktionen
Der RSS-Connector kann als Aktion verwendet werden. Er verfügt über Trigger. Alle Connectors unterstützen Daten im JSON- und XML-Format.

 Der RSS-Connector verfügt über die folgenden Aktionen und/oder Trigger:

### RSS-Aktionen
Sie können diese Aktionen ausführen:

| Aktion | Beschreibung |
| --- | --- |
| [ListFeedItems](connectors-create-api-rss.md#listfeeditems) |Ruft alle RSS-Feedelemente ab. |

### RSS-Trigger
Sie können auf diese Ereignisse lauschen:

| Trigger | Beschreibung |
| --- | --- |
| Wenn ein neues Feedelement veröffentlicht wird |Löst einen Workflow aus, sobald ein neuer Feed veröffentlicht wird. |

## Herstellen einer Verbindung mit RSS
> [!INCLUDE [Schritte zum Herstellen einer Verbindung mit einem RSS-Feed](../../includes/connectors-create-api-rss.md)]
> 
> [!TIP]
> Sie können diese Verbindung in anderen Logik-Apps verwenden.
> 
> 

## Referenz für RSS
Gilt für Version 1.0.

## OnNewFeed
Wenn ein neues Feedelement veröffentlicht wird: Löst einen Workflow aus, sobald ein neuer Feed veröffentlicht wird.

```GET: /OnNewFeed```

| Name | Datentyp | Erforderlich | Enthalten in | Standardwert | Beschreibung |
| --- | --- | --- | --- | --- | --- |
| feedUrl |string |Ja |query |(Keine) |Feed-URL |

#### Antwort
| Name | Beschreibung |
| --- | --- |
| 200 |OK |
| 202 |Zulässig |
| 400 |Ungültige Anforderung |
| 401 |Nicht autorisiert |
| 403 |Verboten |
| 404 |Nicht gefunden |
| 500 |Interner Serverfehler. Unbekannter Fehler aufgetreten |
| default |Fehler beim Vorgang. |

## ListFeedItems
Alle RSS-Feedelemente auflisten: Ruft alle RSS-Feedelemente ab.

```GET: /ListFeedItems```

| Name | Datentyp | Erforderlich | Enthalten in | Standardwert | Beschreibung |
| --- | --- | --- | --- | --- | --- |
| feedUrl |string |Ja |query |(Keine) |Feed-URL |

#### Antwort
| Name | Beschreibung |
| --- | --- |
| 200 |OK |
| 202 |Zulässig |
| 400 |Ungültige Anforderung |
| 401 |Nicht autorisiert |
| 403 |Verboten |
| 404 |Nicht gefunden |
| 500 |Interner Serverfehler. Unbekannter Fehler aufgetreten |
| default |Fehler beim Vorgang. |

## Objektdefinitionen
### TriggerBatchResponse[FeedItem]
| Eigenschaftenname | Datentyp | Erforderlich |
| --- | --- | --- |
| value |array |Nein |

### FeedItem
| Eigenschaftenname | Datentyp | Erforderlich |
| --- | --- | --- |
| id |string |Ja |
| title |string |Ja |
| Inhalt |string |Ja |
| links |array |Nein |
| updatedOn |string |Nein |

## Nächste Schritte
[Erstellen einer Logik-App](../app-service-logic/app-service-logic-create-a-logic-app.md)

<!---HONumber=AcomDC_0824_2016-->