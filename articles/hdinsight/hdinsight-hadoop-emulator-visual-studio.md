---
title: Verwenden der Microsoft Azure Data Lake-Tools für Visual Studio mit der Hortonworks Sandbox | Microsoft Docs
description: Es wird beschrieben, wie Sie die Azure Data Lake-Tools für Visual Studio mit der Hortonworks Sandbox verwenden (Ausführung auf einer lokalen VM). Mit diesen Tools können Sie Hive- und Pig-Aufträge in der Sandbox erstellen und ausführen und die Auftragsausgabe und den Verlauf anzeigen.
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: paulettm
editor: cgronlun

ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 08/26/2016
ms.author: larryfr

---
# Verwenden der Azure Data Lake-Tools für Visual Studio mit der Hortonworks Sandbox
Die Azure Data Lake-Tools für Visual Studio enthalten Tools für die Verwendung von generischen Hadoop-Clustern sowie Tools für Azure Data Lake und HDInsight. Dieses Dokument enthält die erforderlichen Schritte zum Nutzen der Azure Data Lake-Tools mit der Hortonworks Sandbox auf einem lokalen virtuellen Computer.

Mit der Hortonworks Sandbox können Sie Hadoop lokal in Ihrer Entwicklungsumgebung verwenden. Nachdem Sie eine Lösung entwickelt haben und diese bedarfsorientiert bereitstellen möchten, können Sie dann die Umstellung auf einen HDInsight-Cluster durchführen.

## Voraussetzungen
* Hortonworks Sandbox, die auf einem virtuellen Computer in Ihrer Entwicklungsumgebung ausgeführt wird. Dieses Dokument wurde geschrieben und getestet, während die Sandbox in Oracle VirtualBox ausgeführt wurde. Diese wurde gemäß den Informationen im Dokument [Erste Schritte im Hadoop-Ökosystem](hdinsight-hadoop-emulator-get-started.md) konfiguriert.
* Visual Studio 2013 oder 2015 (beliebige Edition).
* [Azure SDK für .NET](https://azure.microsoft.com/downloads/) 2.7.1 oder höher
* [Azure Data Lake-Tools für Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504)

## Konfigurieren von Kennwörtern für die Sandbox
Stellen Sie sicher, dass die Hortonworks Sandbox ausgeführt wird. Führen Sie anschließend die Schritte unter [Erste Schritte im Hadoop-Ökosystem](hdinsight-hadoop-emulator-get-started.md#set-passwords) aus, um das Kennwort für das `root`-SSH-Konto und das `admin`-Ambari-Konto zu konfigurieren. Diese Kennwörter werden verwendet, wenn Sie aus Visual Studio eine Verbindung mit der Sandbox herstellen.

## Verbinden der Tools mit der Sandbox
1. Öffnen Sie Visual Studio, und wählen Sie **Ansicht** und dann **Server-Explorer**.
2. Klicken Sie im **Server-Explorer** mit der rechten Maustaste auf den Eintrag **HDInsight**, und wählen Sie dann die Option **Connect to HDInsight Emulator** (Verbindung mit HDInsight Emulator herstellen).
   
    ![Verbindung mit dem HDInsight-Emulator herstellen](./media/hdinsight-hadoop-emulator-visual-studio/connect-emulator.png)
3. Geben Sie im Dialogfeld **Connect to HDInsight Emulator** (Verbindung mit HDInsight Emulator herstellen) das Kennwort ein, das Sie für Ambari konfiguriert haben.
   
    ![Ambari-Kennwort eingeben](./media/hdinsight-hadoop-emulator-visual-studio/enter-ambari-password.png)
   
    Wählen Sie **Weiter**, um fortzufahren.
4. Verwenden Sie das Feld **Kennwort**, um das Kennwort einzugeben, das Sie für das `root`-Konto konfiguriert haben. Behalten Sie für die anderen Felder den Standardwert bei.
   
    ![root-Kennwort eingeben](./media/hdinsight-hadoop-emulator-visual-studio/enter-root-password.png)
   
    Wählen Sie **Weiter**, um fortzufahren.
5. Warten Sie, bis die Überprüfung der Dienste abgeschlossen ist. Es kann vorkommen, dass die Überprüfung nicht erfolgreich ist und Sie eine Aufforderung zum Aktualisieren der Konfiguration erhalten. Wählen Sie in diesem Fall die Schaltfläche **Aktualisieren**, und warten Sie, bis die Konfiguration und Verifizierung für den Dienst abgeschlossen ist.
   
    ![Fehler und Schaltfläche „Aktualisieren“](./media/hdinsight-hadoop-emulator-visual-studio/fail-and-update.png)
   
   > [!NOTE]
   > Während des Aktualisierungsvorgangs wird Ambari verwendet, um die Hortonworks Sandbox-Konfiguration so zu ändern, dass die Erwartungen der Azure Data Lake-Tools für Visual Studio erfüllt werden.
   > 
   > 
   
    Wählen Sie nach Abschluss der Überprüfung die Option **Fertig stellen**, um die Konfiguration abzuschließen.
   
    ![Verbindungsvorgang beenden](./media/hdinsight-hadoop-emulator-visual-studio/finished-connect.png)
   
   > [!NOTE]
   > Je nach Geschwindigkeit Ihrer Entwicklungsumgebung und der Arbeitsspeichermenge, die dem virtuellen Computer zugewiesen wurde, kann das Konfigurieren und Überprüfen der Dienste mehrere Minuten dauern.
   > 
   > 

Nach dem Ausführen dieser Schritte enthält der Server-Explorer im Abschnitt „HDInsight“ jetzt den Eintrag „HDInsight local cluster“ (Lokaler HDInsight-Cluster).

## Schreiben einer Hive-Abfrage
Hive stellt eine SQL-ähnliche Abfragesprache (HiveQL) für die Arbeit mit strukturierten Daten bereit. Führen Sie die folgenden Schritte aus, um zu erfahren, wie Sie Ad-hoc-Abfragen für den lokalen Cluster ausführen.

1. Klicken Sie im **Server-Explorer** mit der rechten Maustaste auf den Eintrag für den lokalen Cluster, den Sie zuvor hinzugefügt haben, und wählen Sie dann die Option **Write a Hive query** (Hive-Abfrage schreiben).
   
    ![Hive-Abfrage schreiben](./media/hdinsight-hadoop-emulator-visual-studio/write-hive-query.png)
   
    Ein neues Abfragefenster wird geöffnet, in dem Sie eine Abfrage schnell schreiben und an den lokalen Cluster senden können.
2. Geben Sie im neuen Abfragefenster Folgendes ein:
   
        select count(*) from sample_08;
   
    Stellen Sie oben im Abfragefenster sicher, dass die Konfiguration für den lokalen Cluster ausgewählt ist, und wählen Sie anschließend **Senden**. Behalten Sie für die anderen Werte (**Batch** und Servername) die Standardwerte bei.
   
    ![Abfragefenster und Schaltfläche „Senden“](./media/hdinsight-hadoop-emulator-visual-studio/submit-hive.png)
   
    Beachten Sie, dass Sie auch das Dropdownmenü neben **Senden** verwenden können, um **Erweitert** auszuwählen. Es wird ein Dialogfeld geöffnet, in dem Sie beim Senden des Auftrags weitere Optionen angeben können.
   
    ![Erweitertes Senden](./media/hdinsight-hadoop-emulator-visual-studio/advanced-hive.png)
3. Nach dem Absenden der Abfrage wird der Auftragsstatus angezeigt. Hier erhalten Sie Informationen zum Auftrag, während er von Hadoop verarbeitet wird. Der Eintrag **Auftragszustand** zeigt den aktuellen Status des Auftrags an. Der Status wird in regelmäßigen Abständen aktualisiert, oder Sie können das Aktualisierungssymbol verwenden, um den Status manuell zu aktualisieren.
   
    ![Auftragszustand](./media/hdinsight-hadoop-emulator-visual-studio/job-state.png)
   
    Nachdem sich der **Auftragsstatus** in **Beendet** geändert hat, wird ein gerichteter azyklischer Graph (Directed Acyclic Graph, DAG) angezeigt. Hiermit wird der Ausführungspfad beschrieben, der per Tez bestimmt wurde (Standardausführungsmodul für Hive im lokalen Cluster).
   
   > [!NOTE]
   > Tez ist auch die Standardeinstellung, wenn Linux-basierte HDInsight-Cluster verwendet werden. Für Windows-basiertes HDInsight ist dies nicht die Standardeinstellung. Zur Verwendung müssen Sie hierbei am Anfang der Hive-Abfrage die Zeile `set hive.execution.engine = tez;` hinzufügen.
   > 
   > 
   
    Verwenden Sie den Link **Auftragsausgabe**, um die Ausgabe anzuzeigen. In diesem Fall ist dies **823**, also die Anzahl von Zeilen in der Tabelle „sample\_08“. Sie können Diagnoseinformationen zum Auftrag anzeigen, indem Sie die Links **Auftragsprotokoll** und **Download YARN Log** (YARN-Protokoll herunterladen) verwenden.
4. Sie können Hive-Aufträge auch interaktiv ausführen, indem Sie das Feld **Batch** in **Interaktiv** ändern und dann **Ausführen** wählen.
   
    ![Interaktive Abfrage](./media/hdinsight-hadoop-emulator-visual-studio/interactive-query.png)
   
    Das Ausgabeprotokoll, das während der Verarbeitung generiert wurde, wird an das Fenster **HiveServer2 Output** (HiveServer2-Ausgabe) gestreamt.
   
   > [!NOTE]
   > Dies sind die gleichen Informationen, die über den Link **Auftragsprotokoll** verfügbar sind, nachdem ein Auftrag abgeschlossen wurde.
   > 
   > 
   
    ![HiveServer2-Ausgabe](./media/hdinsight-hadoop-emulator-visual-studio/hiveserver2-output.png)

## Erstellen eines Hive-Projekts
Sie können auch ein Projekt erstellen, das mehrere Hive-Skripts enthält. Dies ist hilfreich, wenn Sie über verwandte Skripts verfügen, die zusammenbleiben oder die Sie mit einem Versionskontrollsystem verwalten müssen.

1. Wählen Sie in Visual Studio die Option **Datei**, **Neu** und dann „Projekt“.
2. Erweitern Sie in der Liste mit den Projekten die Option **Vorlagen** > **Azure Data Lake**, und wählen Sie **HIVE (HDInsight)**. Wählen Sie in der Liste mit den Vorlagen die Option **Hive Sample** (Hive-Beispiel) aus. Geben Sie einen Namen und Speicherort ein, und klicken Sie auf **OK**.
   
    ![HIVE (HDInsight)-Vorlage](./media/hdinsight-hadoop-emulator-visual-studio/new-hive-project.png)

Das Projekt **Hive Sample** (Hive-Beispiel) enthält zwei Skripts: **WebLogAnalysis.hql** und **SensorDataAnalysis.hql**. Sie können diese Skripts absenden, indem Sie oben im Fenster jeweils die Schaltfläche **Senden** verwenden.

## Erstellen eines Pig-Projekts
Während für Hive eine SQL-ähnliche Sprache für die Arbeit mit strukturierten Daten bereitgestellt wird, wird für Pig eine Sprache (Pig Latin) verwendet, mit der Sie eine Pipeline mit Transformationen entwickeln können, die auf Ihre Daten angewendet werden. Verwenden Sie die folgenden Schritte, um Pig mit dem lokalen Cluster zu nutzen.

1. Öffnen Sie Visual Studio, und wählen Sie **Datei**, **Neu** und dann **Projekt**. Erweitern Sie in der Liste mit den Projekten die Option **Vorlagen** > **Azure Data Lake**, und wählen Sie **Pig (HDInsight)**. Wählen Sie in der Liste der Vorlagen die Option **Pig-Anwendung** aus. Geben Sie einen Namen und einen Speicherort ein, und klicken Sie auf **OK**.
   
    ![Pig (HDInsight)-Projekt](./media/hdinsight-hadoop-emulator-visual-studio/new-pig.png)
2. Geben Sie Folgendes als Inhalt der Datei **script.pig** ein, die mit diesem Projekt erstellt wurde.
   
        a = LOAD '/demo/data/Website/Website-Logs' AS (
            log_id:int, 
            ip_address:chararray, 
            date:chararray, 
            time:chararray, 
            landing_page:chararray, 
            source:chararray);
        b = FILTER a BY (log_id > 100);
        c = GROUP b BY ip_address;
        DUMP c;
   
    Für Pig wird zwar eine andere Sprache als für Hive verwendet, aber die Ausführung über die Schaltfläche **Senden** ist für beide Sprachen gleich. Wenn Sie die Dropdownliste neben **Senden** auswählen, wird ein erweitertes Sendedialogfeld für Pig angezeigt.
   
    ![Erweitertes Senden (Pig)](./media/hdinsight-hadoop-emulator-visual-studio/advanced-pig.png)
3. Der Auftragsstatus und die Ausgabe werden genauso wie für eine Hive-Abfrage angezeigt.
   
    ![Abbildung mit einem abgeschlossenen Pig-Auftrag](./media/hdinsight-hadoop-emulator-visual-studio/completed-pig.png)

## Anzeigen von Aufträgen
Mit Azure Data Lake-Tools können Sie leicht Informationen zu Aufträgen anzeigen, die mit Hadoop ausgeführt wurden. Verwenden Sie die folgenden Schritte zum Anzeigen der Aufträge, die im lokalen Cluster ausgeführt wurden.

1. Klicken Sie im **Server-Explorer** mit der rechten Maustaste auf den lokalen Cluster, und wählen Sie **Aufträge anzeigen**. Eine Liste mit den Aufträgen wird angezeigt, die an den Cluster übermittelt wurden.
   
    ![Aufträge anzeigen](./media/hdinsight-hadoop-emulator-visual-studio/view-jobs.png)
2. Wählen Sie in der Liste mit den Aufträgen einen Auftrag aus, um die Auftragsdetails anzuzeigen.
   
    ![Auftrag auswählen](./media/hdinsight-hadoop-emulator-visual-studio/view-job-details.png)
   
    Die angezeigten Informationen ähneln der Anzeige nach dem Ausführen einer Hive- oder Pig-Abfrage und enthalten Links zum Anzeigen der Ausgabe und der Protokollinformationen.
3. Hier können Sie den Auftrag auch ändern und erneut senden.

## Anzeigen von Hive-Datenbanken
1. Erweitern Sie im **Server-Explorer** den Eintrag **HDInsight local cluster** (Lokaler HDInsight-Cluster), und erweitern Sie dann die Option **Hive Databases** (Hive-Datenbanken). Die Datenbanken **Standard** und **xademo** werden im lokalen Cluster sichtbar. Wenn Sie eine Datenbank erweitern, werden die in der Datenbank enthaltenen Tabellen angezeigt.
   
    ![Erweiterte Datenbanken](./media/hdinsight-hadoop-emulator-visual-studio/expanded-databases.png)
2. Beim Erweitern einer Tabelle werden die Spalten der Tabelle angezeigt. Sie können mit der rechten Maustaste auf eine Tabelle klicken und die Option **View Top 100 Rows** (Die ersten 100 Zeilen anzeigen) wählen, um die Daten schnell anzuzeigen.
   
    ![Hive-Datenbankansicht](./media/hdinsight-hadoop-emulator-visual-studio/view-100.png)

### Datenbank- und Tabelleneigenschaften
Sie werden vielleicht bemerkt haben, dass Sie für eine Datenbank oder Tabelle die **Eigenschaften** anzeigen können. Die Details zum ausgewählten Element werden im Eigenschaftenfenster aufgeführt.

![Eigenschaften](./media/hdinsight-hadoop-emulator-visual-studio/properties.png)

### Erstellen einer Tabelle
Klicken Sie zum Erstellen einer neuen Tabelle mit der rechten Maustaste auf eine Datenbank, und wählen Sie dann **Tabelle erstellen**.

![Tabelle erstellen](./media/hdinsight-hadoop-emulator-visual-studio/create-table.png)

Sie können die Tabelle dann mit einem Formular erstellen. Der unformatierte HiveQL-Code, der zum Erstellen der Tabelle verwendet wird, wird unten auf der Seite angezeigt.

![Tabelle erstellen – Formular](./media/hdinsight-hadoop-emulator-visual-studio/create-table-form.png)

## Nächste Schritte
* [Learning the ropes of the Hortonworks Sandbox](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/) (Einführung in Hortonworks Sandbox)
* [Hadoop tutorial - Getting started with HDP](http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/) (Hadoop-Tutorial – Erste Schritte mit HDP)

<!---HONumber=AcomDC_0921_2016-->