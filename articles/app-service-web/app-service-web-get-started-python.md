---
title: "Bereitstellen Ihrer ersten Python-Web-App für Azure in fünf Minuten | Microsoft Docs"
description: "Erfahren Sie, wie einfach die Ausführung von Web-Apps in App Service ist, indem Sie eine Beispiel-App bereitstellen. Sie können in kürzester Zeit mit der Entwicklung beginnen und sofort Ergebnisse erzielen."
services: app-service\web
documentationcenter: 
author: cephalin
manager: wpickett
editor: 
ms.assetid: 928ee2e5-6143-4c0c-8546-366f5a3d80ce
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 10/13/2016
ms.author: cephalin
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: d0614b80ca8be3e37b0168ddafc110f1adaad123


---
# <a name="deploy-your-first-python-web-app-to-azure-in-five-minutes"></a>Bereitstellen Ihrer ersten Python-Web-App für Azure in fünf Minuten
In diesem Tutorial erfahren Sie, wie Sie Ihre erste Python-Web-App für [Azure App Service](../app-service/app-service-value-prop-what-is.md)bereitstellen.
Mit App Service können Sie Web-Apps, [Mobile App-Back-Ends](/documentation/learning-paths/appservice-mobileapps/) und [API-Apps](../app-service-api/app-service-api-apps-why-best-platform.md) erstellen.

In diesem Tutorial führen Sie folgende Schritte aus: 

* Erstellen einer Web-App in Azure App Service
* Bereitstellen von Python-Beispielcode
* Anzeigen des live in der Produktion ausgeführten Codes
* Aktualisieren Ihrer Web-App wie beim [Ausführen von Git-Commits mittels Push](https://git-scm.com/docs/git-push)

## <a name="prerequisites"></a>Voraussetzungen
* [Git](http://www.git-scm.com/downloads).
* [Azure-Befehlszeilenschnittstelle](../xplat-cli-install.md).
* Ein Microsoft Azure-Konto. Falls Sie noch kein Konto haben, können Sie sich [für eine kostenlose Testversion registrieren](/pricing/free-trial/?WT.mc_id=A261C142F) oder [Ihre Visual Studio-Abonnentenvorteile aktivieren](/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F).

> [!NOTE]
> Zum [Testen von App Service](http://go.microsoft.com/fwlink/?LinkId=523751) benötigen Sie kein Azure-Konto. Sie können eine Starter-App erstellen und bis zu einer Stunde damit experimentieren – ohne Kreditkarte und ohne jegliche Verpflichtungen.
> 
> 

## <a name="deploy-a-python-web-app"></a>Bereitstellen einer Python-Web-App
1. Öffnen Sie eine neue Windows-Eingabeaufforderung, ein PowerShell-Fenster, eine Linux-Shell oder ein OS X-Terminal. Vergewissern Sie sich durch Ausführen von `git --version` und `azure --version`, dass Git und die Azure-Befehlszeilenschnittstelle auf Ihrem Computer installiert sind.
   
    ![Testen der Installation der CLI-Tools für Ihre erste Web-App in Azure](./media/app-service-web-get-started/1-test-tools.png)
   
    Entsprechende Downloadlinks finden Sie bei Bedarf unter [Voraussetzungen](#Prerequisites) .
2. Melden Sie sich wie folgt bei Azure an:
   
        azure login
   
    Folgen Sie der Hilfemeldung, um den Anmeldeprozess fortzusetzen.
   
    ![Anmelden bei Azure zur Erstellung Ihrer ersten Web-App](./media/app-service-web-get-started/3-azure-login.png)
3. Wechseln Sie Azure-CLI in den ASM-Modus, und legen Sie dann den Bereitstellungsbenutzer für App Service fest. Sie werden später unter Verwendung der Anmeldeinformationen Code bereitstellen.
   
        azure config mode asm
        azure site deployment user set --username <username> --pass <password>
4. Wechseln Sie in ein Arbeitsverzeichnis (`CD`), und klonen Sie die Beispiel-App wie folgt:
   
        git clone https://github.com/Azure-Samples/app-service-web-python-get-started.git
5. Wechseln Sie in das Repository Ihrer Beispiel-App. Beispiel:
   
        cd app-service-web-python-get-started
6. Erstellen Sie die App Service-App-Ressource in Azure mit einem eindeutigen App-Namen und dem zuvor konfigurierten Bereitstellungsbenutzer. Geben Sie die Zahl der gewünschten Region ein, wenn Sie dazu aufgefordert werden.
   
        azure site create <app_name> --git --gitusername <username>
   
    ![Erstellen der Azure-Ressource für Ihre erste Web-App in Azure](./media/app-service-web-get-started-languages/python-site-create.png)
   
    Ihre App wird jetzt in Azure erstellt. Ihr aktuelles Verzeichnis wird außerdem für Git initialisiert und mit der neuen App Service-App als Git-Remoteelement verbunden.
    Sie können zwar auch zur URL der App (http://&lt;app_name>.azurewebsites.net) navigieren, um die HTML-Standardseite anzuzeigen, wir möchten hier aber eigentlich Ihren eigenen Code verwenden.
7. Gehen Sie wie bei anderen Pushübertragungen von Code mit Git vor, um Ihren Beispielcode für die Azure-App bereitzustellen. Wenn Sie aufgefordert werden, verwenden Sie das Kennwort, das Sie zuvor konfiguriert haben.
   
        git push azure master
   
    ![Übertragen von Code an Ihre ersten Web-App in Azure mittels Push](./media/app-service-web-get-started-languages/python-git-push.png)
   
    Mit `git push` wird nicht nur Code in Azure eingefügt, sondern es werden auch Bereitstellungsaufgaben im Bereitstellungsmodul ausgelöst. 
    Falls Ihr Projektstamm (Repositorystamm) eine Datei mit dem Namen „requirements.txt“ (Python) enthält, stellt das Bereitstellungsskript die erforderlichen Pakete wieder für Sie her. 

Herzlichen Glückwunsch! Sie haben Ihre App in Azure App Service bereitgestellt.

## <a name="see-your-app-running-live"></a>Verfolgen der Liveausführung der App
Führen Sie den folgenden Befehl aus einem beliebigen Verzeichnis in Ihrem Repository aus, um die Liveausführung der App in Azure zu verfolgen:

    azure site browse

## <a name="make-updates-to-your-app"></a>Durchführen von Updates für die App
Sie können jetzt Git verwenden, um aus Ihrem Projektstamm (Repositorystamm) jederzeit einen Pushvorgang durchzuführen und so ein Update für die Live-Website vorzunehmen. Dazu gehen Sie wie bei der erstmaligen Bereitstellung Ihres Codes vor. Wenn Sie beispielsweise eine neue Änderung übertragen möchten, die Sie lokal getestet haben, führen Sie einfach die folgenden Befehle in Ihrem Projektstamm (Repositorystamm) aus:

    git add .
    git commit -m "<your_message>"
    git push azure master

## <a name="next-steps"></a>Nächste Schritte
[Erstellen, Konfigurieren und Bereitstellen einer Django-Web-App für Azure in Visual Studio](web-sites-python-ptvs-django-mysql.md) In diesem Tutorial eignen Sie sich grundlegende Kenntnisse zum Ausführen einer Python-Web-App in Azure an. Sie erhalten u.a. Informationen zu den folgenden Schritten:

* Erstellen und Bereitstellen einer Python-App mithilfe einer Vorlage
* Festlegen der Python-Version
* Erstellen virtueller Umgebungen
* Herstellen einer Verbindung mit einer Datenbank

Sie können auch weiter mit Ihrer ersten Web-App arbeiten. Beispiel:

* Testen Sie [weitere Methoden zum Bereitstellen Ihres Codes in Azure](web-sites-deploy.md). Wenn Sie eines Ihrer GitHub-Repositorys zur Bereitstellung verwenden möchten, wählen Sie beispielsweise unter **Bereitstellungsoptionen** einfach **GitHub** anstelle von **Lokales Git-Repository** aus.
* Entwickeln Sie Ihre Azure-App weiter. Authentifizieren Sie Ihre Benutzer. Skalieren Sie die App je nach Bedarf. Richten Sie einige Leistungswarnungen ein. Es sind jeweils nur wenige Klicks erforderlich. Weitere Informationen finden Sie unter [Hinzufügen von Funktionen zu Ihrer ersten Web-App](app-service-web-get-started-2.md).




<!--HONumber=Nov16_HO2-->


