---
title: "Was ist SQL Database? Einführung in SQL-Datenbank | Microsoft Docs"
description: 'Get an introduction to SQL Database: technical details and capabilities of Microsoft''s relational database management system (RDBMS) in the cloud.'
keywords: "Einführung in SQL, Was ist SQL-Datenbank?"
services: sql-database
documentationcenter: 
author: shontnew
manager: jhubbard
editor: cgronlun
ms.assetid: c561f600-a292-4e3b-b1d4-8ab89b81db48
ms.service: sql-database
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: data-management
ms.date: 11/08/2016
ms.author: shkurhek
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 23d68a523fa9a79d2b5154eef04a5f6e706bdeb2


---
# <a name="what-is-sql-database-introduction-to-sql-database"></a>Was ist SQL Database? Einführung in SQL-Datenbank
SQL-Datenbank ist ein relationaler Datenbankdienst in der Cloud, der auf dem marktführenden Microsoft SQL Server-Modul basiert und unternehmenswichtige Funktionen bietet. SQL-Datenbank bietet vorhersagbare Leistung, Skalierbarkeit ohne Ausfallzeiten, Geschäftskontinuität, Datenschutz und nahezu keinerlei Verwaltungsaufwand. Anstatt auf die Verwaltung virtueller Computer und der Infrastruktur können Sie sich auf die schnelle Entwicklung von Apps und eine beschleunigte Markteinführung konzentrieren. Da SQL-Datenbank auf dem [SQL Server](https://msdn.microsoft.com/library/bb545450.aspx) -Modul basiert, werden vorhandene SQL Server-Tools, -Bibliotheken und -APIs unterstützt, sodass diese einfacher in die Cloud verschoben und auf die Cloud ausgedehnt werden können.

Dieser Artikel bietet eine Einführung in die wesentlichen Konzepte und Features von SQL-Datenbank im Hinblick auf Leistung, Skalierbarkeit und Verwaltbarkeit sowie Links zu detaillierten Informationen. Wenn Sie sich damit vertraut gemacht haben, können Sie binnen Minuten [eine erste SQL-Datenbank erstellen](sql-database-get-started.md) oder [einen Pool für elastische Datenbanken erstellen](sql-database-elastic-pool-create-portal.md). Wenn Sie eingehendere Informationen benötigen, finden Sie diese in diesem 30 Minuten langen Video.

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON326/player]
> 
> 

## <a name="adjust-performance-and-scale-without-downtime"></a>Anpassen von Leistung und Skalierung ohne Ausfallzeiten
Für SQL-Datenbanken sind die *Dienstebenen*Basic, Standard und Premium verfügbar. Jede Dienstebene bietet [verschiedene Leistungsstufen und Funktionen](sql-database-service-tiers.md) zur Unterstützung geringer und hoher Datenbankworkloads. Sie können Ihre erste App mit einer kleinen Datenbank zu einem geringen Preis pro Monat erstellen und dann jederzeit manuell oder programmgesteuert [die Dienstebene ändern](sql-database-scale-up.md) , wenn Ihre App weltweit genutzt wird, ohne dass Ausfallzeiten für die App oder Ihre Kunden entstehen.

Für viele Unternehmen und Apps genügt es, wenn Datenbanken erstellt werden können und sich die Leistung von Einzeldatenbanken nach oben oder unten anpassen lässt, insbesondere, wenn die Nutzungsmuster relativ vorhersagbar sind. Bei unvorhersagbaren Nutzungsmustern kann es jedoch schwer sein, die Kosten und Ihr Geschäftsmodell zu verwalten.

[elastischen Pools](sql-database-elastic-pool.md) in SQL-Datenbank beheben. Das Konzept ist denkbar einfach. Sie weisen einem Pool eine bestimmte Leistung zu und zahlen für die gesamte Leistung des Pools und nicht für die Leistung der Einzeldatenbanken. Sie müssen die Datenbankleistung nicht nach oben oder unten anpassen. Die Datenbanken in dem Pool, die sogenannten *elastischen Datenbanken*, werden automatisch nach Bedarf zentral hoch- und herunterskaliert. Elastische Datenbanken nutzen die Grenzwerte des Pools, überschreiten sie jedoch nicht, sodass Ihre Kosten vorhersagbar bleiben, selbst wenn sich die Nutzung der Datenbanken nicht prognostizieren lässt. Darüber hinaus können Sie [Datenbanken im Pool hinzufügen und entfernen](sql-database-elastic-pool-manage-portal.md)und so Ihre App von einigen wenigen auf Tausende Datenbanken skalieren, und das in einem kontrollierten Kostenrahmen. Weitere Informationen zu Entwurfsmustern für SaaS-Anwendungen, für die elastische Pools verwendet werden, finden Sie unter [Entwurfsmuster für SaaS-Anwendungen mit mehreren Mandanten und Azure SQL-Datenbank](sql-database-design-patterns-multi-tenancy-saas-applications.md).

Ob Sie sich nun für Einzeldatenbanken oder elastische Datenbanken entscheiden, ist dies keine endgültige Entscheidung. Sie können Einzeldatenbanken mit Pools für elastische Datenbanken kombinieren und die Dienstebenen von Einzeldatenbanken und Pools ändern, um innovative Anwendungsdesigns zu erhalten. Außerdem können Sie mit der Leistungsstärke und Reichweite von Azure einzelne Azure-Dienste mit der SQL-Datenbank kombinieren, um Ihre individuellen Anforderungen an modernes App-Design zu erfüllen, Kosten- und Ressourceneffizienz zu fördern und neue Geschäftsmöglichkeiten zu erschließen.

Aber wie können Sie nun die relative Leistung von Datenbanken und Datenbankpools vergleichen? Wie entscheiden Sie sich beim Anpassen für die richtige Lösung? Die Antwort liegt in den Datenbank-Transaktionseinheiten (Database Transaction Unit, DTU) für Einzeldatenbanken und in den elastischen DTUs (eDTUs) für elastische Datenbanken und Datenbankpools. Ausführliche Informationen finden Sie unter [SQL-Datenbankoptionen und -leistung: Grundlegendes zum Angebot in den einzelnen Tarifen](sql-database-service-tiers.md) .

## <a name="keep-your-app-and-business-running"></a>Aufrechterhalten von App-Ausführung und Geschäftskontinuität
Durch die Unterstützung eines globalen Netzwerks von Microsoft-verwalteten Rechenzentren stellt die in der Branche führende Verfügbarkeit von Azure mit einer Vereinbarung zum Servicelevel [(SLA)](http://azure.microsoft.com/support/legal/sla/) von 99,99% sicher, dass Ihre Apps rund um die Uhr ausgeführt werden. Mit jeder SQL-Datenbank nutzen Sie den integrierten Datenschutz und die Fehlertoleranz, die Sie andernfalls entwerfen, kaufen, erstellen und verwalten müssten. Abhängig von Ihren Geschäftsanforderungen benötigen Sie möglicherweise dennoch weitere Schutzebenen, um sicherzustellen, dass Ihre App und Ihr Geschäftsbetrieb bei einem Notfall, einem Fehler o. Ä. schnell wiederhergestellt werden kann. Bei der SQL-Datenbank bietet jede Dienstebene verschiedene Funktionen, mit denen Sie schnell einsatzbereit sind (und bleiben). Über die Point-in-Time-Wiederherstellung können Sie den früheren Status einer Datenbank für bis zu 35 Tage wiederherstellen. Bei einem Ausfall des Rechenzentrums, das Ihre Datenbanken hostet, können Sie darüber hinaus Datenbankreplikate in einer anderen Region nutzen. Zudem können Sie Replikate für Aktualisierungen oder zur Verlegung in andere Regionen verwenden.

![SQL-Datenbank-Georeplikation](./media/sql-database-technical-overview/azure_sqldb_map.png)

Ausführliche Informationen zu den unterschiedlichen Funktionen der Geschäftskontinuität bei den verschiedenen Dienstebenen finden Sie unter [Übersicht über die Geschäftskontinuität](sql-database-business-continuity.md) .

## <a name="secure-your-data"></a>Sichern der Daten
SQL Server bietet traditionell eine solide Datensicherheit, die SQL-Datenbank mit Funktionen zur Zugriffseinschränkung, zum Datenschutz und zur Überwachung von Aktivitäten fortführt. Eine kurze Übersicht über die Sicherheitsoptionen in SQL-Datenbank finden Sie unter [Sichern der SQL-Datenbank](sql-database-security.md) . Ausführlichere Informationen zu den Sicherheitsfunktionen finden Sie unter [Sicherheitscenter für SQL Server-Datenbankmodul und Azure SQL-Datenbank](https://msdn.microsoft.com/library/bb510589) . Schließlich finden Sie im [Azure Trust Center](https://azure.microsoft.com/support/trust-center/security/) Informationen zur Sicherheit der Azure-Plattform.

## <a name="next-steps"></a>Nächste Schritte
Nachdem Sie eine Einführung in SQL-Datenbank gelesen und die Frage „Was ist SQL-Datenbank?“ beantwortet haben, sind Sie bereit für Folgendes:

* Unter [Preise](https://azure.microsoft.com/pricing/details/sql-database/) finden Sie Preisvergleiche und Rechner für Einzeldatenbanken und elastische Datenbanken.
* Erfahren Sie mehr über [elastische Pools](sql-database-elastic-pool.md).
* Beginnen Sie mit dem [Erstellen einer ersten Azure SQL-Datenbank](sql-database-get-started.md).
* [Verbinden und Abfragen mit SSMS](sql-database-connect-query-ssms.md)
* Erstellen Sie Ihre erste App in C#, Java, Node.js, PHP, Python oder Ruby: [Verbindungsbibliotheken für SQL-Datenbank und SQL Server](sql-database-libraries.md)
* Einen Index mit den Titeln und Beschreibungen finden Sie unter [Alle Themen für den Azure SQL-Datenbankdienst](sql-database-index-all-articles.md).




<!--HONumber=Nov16_HO2-->


