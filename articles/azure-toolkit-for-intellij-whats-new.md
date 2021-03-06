---
title: Neuerungen im Azure-Toolkit für IntelliJ | Microsoft Docs
description: Hier können Sie sich über die neuesten Features des Azure-Toolkits für IntelliJ informieren.
services: ''
documentationcenter: java
author: rmcmurray
manager: erikre
editor: ''

ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: Java
ms.topic: article
ms.date: 11/01/2016
ms.author: robmcm;asirveda;martinsawicki

---
# <a name="whats-new-in-the-azure-toolkit-for-intellij"></a>Neuerungen im Azure-Toolkit für IntelliJ
## <a name="azure-toolkit-for-intellij-releases"></a>Versionen des Azure-Toolkits für IntelliJ
Dieser Artikel enthält Informationen zu den verschiedenen Versionen und den neuesten Updates des Azure-Toolkits für IntelliJ.

> [!NOTE]
> Es gibt auch ein Azure-Toolkit für die Eclipse-IDE. Weitere Informationen finden Sie unter [Azure-Toolkit für Eclipse].
> 
> 

### <a name="august-26-2016"></a>26. August 2016
Das Release des Azure-Toolkits für IntelliJ vom April 2016 umfasst folgende Erweiterungen:

* **Benutzerdefinierte JDK-Distributionen**. Das Azure-Toolkit für IntelliJ unterstützt jetzt das Angeben und Bereitstellen einer beliebigen JDK-Version in Ihrem Azure-WebApp-Container:
  * Zusätzlich zu den von Azure bereitgestellten JDKs können Sie aus einem umfassenden Angebot an Zulu OpenJDK-Versionen auswählen, die von Azul Systems für Azure zur Verfügung gestellt wurden.
  * Sie können auch eine eigene JDK-Distribution angeben, wenn Sie diese als ZIP-Datei in Ihr Speicherkonto hochladen.
* **Verbesserungen der Azure Explorer-Ansicht**:
  * Unterstützung für die Verwaltung virtueller Computer über das neue Resource Manager-Modell von Azure: Sie können auf Resource Manager basierende virtuelle Computer auflisten, erstellen und löschen, ohne die IDE verlassen zu müssen.
  * Unterstützung für die Blobverwaltung im Speicherkonto über Azure Resource Manager, zur Ergänzung der vorhandenen Funktionalität für die Verwaltung „klassischer“ Speicherkonten.
* **Microsoft JDBC Driver 6.0 für SQL Server**. Dieses Update enthält den neuesten JDBC-Treiber für Microsoft SQL Server (Version 6.0) – dieser wird jetzt als Bibliothek bereitgestellt, die Sie ganz einfach zu Ihren Java-Projekten hinzufügen können. Die ältere Version wird hierdurch ersetzt.

### <a name="june-29-2016"></a>29. Juni 2016
Die Version des Azure-Toolkits für IntelliJ vom Juni 2016 umfasst folgende Erweiterungen:

* **Java 8 erforderlich**. Für das Azure-Toolkit für IntelliJ ist nun Java 8 erforderlich, was allerdings nur für das Toolkit gilt. Ihre Anwendungen können weiterhin alle Java-Versionen verwenden, die von Azure unterstützt werden.
* **Unterstützung der neuesten Java JDKs**. Die neuesten Versionen der Java JDKs werden jetzt vom Azure-Toolkit für IntelliJ unterstützt.
* **Unterstützung des Azure SDK, Version 2.9.1**. Die neueste Version des Azure SDK ist jetzt die Mindestvoraussetzung für das Azure-Toolkit für IntelliJ.
* **Integrierte Beispiele**. Das Azure-Toolkit für IntelliJ bietet jetzt mehrere Beispielanwendungen, die Entwicklern den Einstieg erleichtern sollen.
* **Integration von HDInsight-Tools**. Die Azure HDInsight-Tools stehen nun im Paket mit dem Azure-Toolkit für IntelliJ zur Verfügung. Weitere Informationen finden Sie unter [Verwenden des HDInsight-Tools-Plug-Ins für IntelliJ IDEA zum Erstellen von Spark-Anwendungen für HDInsight Spark-Cluster unter Linux].
* **Remotedebuggen von Java-Web-Apps**. Das Azure-Toolkit für IntelliJ unterstützt jetzt das Remotedebuggen von Java-Web-Apps in Azure App Service.

### <a name="april-12-2016"></a>12. April 2016
Die Version des Azure-Toolkits für IntelliJ vom April 2016 umfasst folgende Erweiterungen:

* **Unterstützung des Azure SDK, Version 2.9.0**. Die neueste Version des Azure SDK ist jetzt die Mindestvoraussetzung für das Azure-Toolkit für IntelliJ.
* **Verschiedene Verbesserungen bei der Benutzerfreundlichkeit, Reaktionsfähigkeit und Leistung in Bezug auf die Unterstützung von Azure-Web-Apps**. Eine Reihe von Leistungsoptimierungen bei der Kommunikation zwischen Toolkit und Azure führen zu einer besseren Reaktionsfähigkeit der Benutzeroberfläche.
* **Möglichkeit zum Löschen eines vorhandenen Webanwendungscontainers in Azure direkt aus IntelliJ heraus**. Mit dem Azure-Toolkit für IntelliJ können Sie jetzt einen vorhandenen Azure-Webcontainer löschen, ohne IntelliJ zu verlassen.

## <a name="see-also"></a>Weitere Informationen
Weitere Informationen zu den Azure-Toolkits für Java-IDEs finden Sie unter den folgenden Links:

* [Azure-Toolkit für Eclipse]
  * [Installieren des Azure-Toolkits für Eclipse]
  * [Erstellen einer „Hello World“-Web-App für Azure in Eclipse]
  * [Neuerungen im Azure-Toolkit für Eclipse]
* [Azure Toolkit für IntelliJ]
  * [Installieren des Azure Toolkit für IntelliJ]
  * [Erstellen einer „Hello World“-Web-App für Azure in IntelliJ]
  * *Neuerungen im Azure-Toolkit für IntelliJ (dieser Artikel)*

Weitere Informationen zum Verwenden von Azure mit Java finden Sie im [Azure Java Developer Center].

<!-- URL List -->

[Azure-Toolkit für Eclipse]: ./azure-toolkit-for-eclipse.md
[Azure Toolkit für IntelliJ]: ./azure-toolkit-for-intellij.md
[Erstellen einer „Hello World“-Web-App für Azure in Eclipse]: ./app-service-web/app-service-web-eclipse-create-hello-world-web-app.md
[Erstellen einer „Hello World“-Web-App für Azure in IntelliJ]: ./app-service-web/app-service-web-intellij-create-hello-world-web-app.md
[Installieren des Azure-Toolkits für Eclipse]: ./azure-toolkit-for-eclipse-installation.md
[Installieren des Azure Toolkit für IntelliJ]: ./azure-toolkit-for-intellij-installation.md
[Neuerungen im Azure-Toolkit für Eclipse]: ./azure-toolkit-for-eclipse-whats-new.md
[Neuerungen im Azure-Toolkit für IntelliJ]: ./azure-toolkit-for-intellij-whats-new.md

[Azure Java Developer Center]: http://go.microsoft.com/fwlink/?LinkID=699547

[Verwenden des HDInsight-Tools-Plug-Ins für IntelliJ IDEA zum Erstellen von Spark-Anwendungen für HDInsight Spark-Cluster unter Linux]: ./hdinsight/hdinsight-apache-spark-intellij-tool-plugin.md



<!--HONumber=Oct16_HO2-->


