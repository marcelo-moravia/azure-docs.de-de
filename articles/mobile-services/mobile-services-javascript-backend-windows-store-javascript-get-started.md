---
title: Erste Schritte mit Mobile Services für Windows Store-JavaScript-Apps | Microsoft Docs
description: Befolgen Sie dieses Lernprogramm für die ersten Schritte bei der Verwendung von Azure Mobile Services für die Windows Store-Entwicklung in JavaScript.
services: mobile-services
documentationcenter: windows
author: ggailey777
manager: erikre
editor: ''

ms.service: mobile-services
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows-store
ms.devlang: javascript
ms.topic: get-started-article
ms.date: 07/21/2016
ms.author: glenga

---
# Erste Schritte mit Mobile Services
[!INCLUDE [mobile-services-selector-get-started](../../includes/mobile-services-selector-get-started.md)]

&nbsp;

[!INCLUDE [mobile-service-note-mobile-apps](../../includes/mobile-services-note-mobile-apps.md)]

> Informationen für die entsprechende Mobile Apps-Version dieses Themas finden Sie unter [Erstellen einer Windows-App mit Mobile Apps](../app-service-mobile/app-service-mobile-windows-store-dotnet-get-started.md).
> 
> 

Dieses Lernprogramm zeigt Ihnen, wie Sie einen cloudbasierten Back-End-Dienst zu einer Windows Store-JavaScript-App mithilfe von Azure Mobile Services hinzufügen. In diesem Lernprogramm erstellen Sie einen neuen mobilen Dienst und eine einfache *To-Do-Listen*-App, die App-Daten im neuen mobilen Dienst speichert. Der von Ihnen zu erstellende mobile Dienst verwendet JavaScript für die serverseitige Geschäftslogik.

Für dieses Lernprogramm benötigen Sie Folgendes:

* Ein aktives Azure-Konto. Wenn Sie noch kein Konto haben, können Sie in nur wenigen Minuten ein kostenloses Testkonto erstellen. Ausführliche Informationen finden Sie unter [Kostenlose Azure-Testversion](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A0E0E5C02&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fdocumentation%2Farticles%2Fmobile-services-javascript-backend-windows-store-javascript-get-started%2F).
* [Visual Studio 2013 Express für Windows]

## Erstellen eines neuen mobilen Diensts
[!INCLUDE [mobile-services-create-new-service](../../includes/mobile-services-create-new-service.md)]

## Erstellen einer neuen Windows Store-App
Sobald Sie Ihren mobilen Dienst erstellt haben, können Sie einer Schnellstartanleitung im klassischen Azure-Portal folgen, um eine neue Windows Store 8.1-JavaScript-App zu erstellen, die eine Verbindung mit Ihrem mobilen Dienst herstellt.

1. Klicken Sie im [klassischen Azure-Portal] auf **Mobile Services** und anschließend auf den mobilen Dienst, den Sie gerade erstellt haben.
2. Klicken Sie auf der Schnellstartregisterkarte unter **Plattform auswählen** auf **Windows**. Erweitern Sie dann **Neue Windows Store-App erstellen**.
3. Laden Sie [Visual Studio 2013][Visual Studio 2013 Express for Windows] auf Ihren lokalen Computer oder virtuellen Computer herunter und installieren Sie es, falls noch nicht geschehen.
4. Klicken Sie auf **TodoItem-Tabelle erstellen**, um eine Tabelle zum Speichern von App-Daten zu erstellen.
5. Wählen Sie unter **App herunterladen und ausführen** eine Sprache für die App aus. Klicken Sie dann auf **Herunterladen**.
   
      Das Projekt für die beispielhafte *To-Do-Listen*-Anwendung, die mit dem mobilen Dienst verbunden ist, wird heruntergeladen. Speichern Sie die komprimierte Projektdatei auf dem lokalen Computer und merken Sie sich, wo Sie sie gespeichert haben.

## Ausführen der Windows-App
Der letzte Schritt dieses Lernprogramms besteht im Erstellen und Ausführen der neuen App.

1. Navigieren Sie zu dem Verzeichnis, in dem Sie die komprimierten Projektdateien gespeichert haben, erweitern Sie die Dateien auf Ihrem Computer, und öffnen Sie die Projektmappen-Datei in Visual Studio.
2. Drücken Sie die **F5**-Taste, um das Projekt neu zu erstellen und die App zu starten.
3. Geben Sie in der App einen beschreibenden Text ein, zum Beispiel *Lernprogramm abschließen*, unter **TodoItem einfügen**. Klicken Sie dann auf **Speichern**.
   
       Dadurch wird eine POST-Anforderung an den neuen, in Azure gehosteten mobilen Dienst gesendet. Daten von der Anforderung werden in die TodoItem-Tabelle eingefügt. In der Tabelle gespeicherte Werte werden vom mobilen Dienst zurückgegeben, und die Daten werden in der zweiten Spalte der App angezeigt.
4. (Optional) Führen Sie die App erneut aus, und beachten Sie, dass nach dem Starten der App Daten, die im vorhergehenden Schritt gespeichert wurden, aus dem mobilen Dienst geladen werden.
5. Zurück im [klassischen Azure-Portal] klicken Sie auf die Registerkarte **Daten** und dann auf die Tabelle **TodoItems**.
   
       Nun können Sie die von der App in die Tabelle eingefügten Daten durchsuchen.

> [!NOTE]
> Sie können den Code, der zur Abfrage und zum Einfügen von Daten auf Ihren mobilen Dienst zugreift, in der Datei "default.js" finden und prüfen.
> 
> 

## Nächste Schritte
Da Sie jetzt den Schnellstart abgeschlossen haben, können Sie anschließend erfahren, wie Sie mit dem [Mobile Services-Client für HTML/JavaScript](mobile-services-html-how-to-use-client-library.md) arbeiten.

[!INCLUDE [app-service-disqus-feedback-slug](../../includes/app-service-disqus-feedback-slug.md)]

<!-- Anchors. -->
[Getting started with Mobile Services]: #getting-started
[Create a new mobile service]: #create-new-service
[Define the mobile service instance]: #define-mobile-service-instance
[Next Steps]: #next-steps

<!-- Images. -->

<!-- URLs. -->
[Visual Studio 2013 Express for Windows]: http://go.microsoft.com/fwlink/?LinkId=257546
[Visual Studio 2013 Express für Windows]: http://go.microsoft.com/fwlink/?LinkId=257546
[Mobile Services SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[klassischen Azure-Portal]: https://manage.windowsazure.com/

<!---HONumber=AcomDC_0727_2016-->