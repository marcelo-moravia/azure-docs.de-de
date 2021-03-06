---
title: Verarbeiten von D2C-Nachrichten mit IoT Hub (.Net) | Microsoft Docs
description: In diesem Tutorial werden nützliche Verfahren zum Verarbeiten von Gerät-zu-Cloud-Nachrichten (Device-to-Cloud, D2C) mit IoT Hub beschrieben.
services: iot-hub
documentationcenter: .net
author: dominicbetts
manager: timlt
editor: ''

ms.service: iot-hub
ms.devlang: csharp
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/05/2016
ms.author: dobett

---
# <a name="tutorial:-how-to-process-iot-hub-device-to-cloud-messages-using-.net"></a>Tutorial: Verarbeiten von D2C-Nachrichten mit IoT Hub mithilfe von .Net
[!INCLUDE [iot-hub-selector-process-d2c](../../includes/iot-hub-selector-process-d2c.md)]

## <a name="introduction"></a>Einführung
Azure IoT Hub ist ein vollständig verwalteter Dienst, der eine zuverlässige und sichere bidirektionale Kommunikation zwischen Millionen von IoT-Geräten und einem Anwendungs-Back-End ermöglicht. Weitere Tutorials ([Erste Schritte mit IoT Hub] und [Senden von C2D-Nachrichten mit IoT Hub][lnk-c2d]) veranschaulichen, wie Sie die grundlegenden Gerät-zu-Cloud- und Cloud-zu-Gerät-Messagingfunktionen von IoT Hub verwenden.

Dieses Tutotrial baut auf dem Code des Tutotrials [Erste Schritte mit IoT Hub] auf und veranschaulicht zwei skalierbare Muster, die Sie zum Verarbeiten von D2C-Nachrichten verwenden können:

* Zuverlässige Speicherung von D2C-Nachrichten in [Azure Blob Storage]. Ein häufiges Szenario ist die *Cold Path*-Analyse, in der Sie Telemetriedaten in Blobs speichern, um sie als Eingabe für Analyseprozesse verwenden zu können. Diese Prozesse können von Tools wie z.B. [Azure Data Factory] oder dem [HDInsight (Hadoop)]-Stapel gesteuert werden.
* Zuverlässige Verarbeitung von *interaktiven* D2C-Nachrichten. D2C-Nachrichten sind interaktiv, wenn es sich um unmittelbare Auslöser für eine Reihe von Aktionen im Anwendungs-Back-End handelt. Beispielsweise könnte ein Gerät eine Alarmnachricht senden, die das Einfügen eines Tickets in ein CRM-System auslöst. Im Gegensatz dazu werden *Datenpunktnachrichten* einfach in ein Analysemodul eingegeben. Temperaturtelemetriedaten von einem Gerät, die zur späteren Analyse gespeichert werden, sind beispielsweise Datenpunktnachrichten.

Da in IoT Hub ein [Event Hubs][lnk-event-hubs]-kompatibler Endpunkt zum Empfangen von D2C-Nachrichten verfügbar gemacht wird, wird in diesem Tutorial eine [EventProcessorHost]-Instanz verwendet. Diese Instanz:

* Zuverlässiges Speichern von *Datenpunkt* -Nachrichten in Azure-Blobspeicher.
* Leitet *interaktive* D2C-Nachrichten zur sofortigen Verarbeitung an eine Azure [Service Bus-Warteschlange] weiter.

Service Bus stellt die zuverlässige Verarbeitung von interaktiven Nachrichten sicher, da Checkpoints pro Nachricht und die Deduplizierung auf Grundlage von Zeitfenstern bereitgestellt werden.

> [!NOTE]
> Eine **EventProcessorHost** -Instanz ist nur eine Möglichkeit, interaktive Nachrichten zu verarbeiten. Weitere Optionen sind [Azure Service Fabric][lnk-service-fabric] und [Azure Stream Analytics][lnk-stream-analytics].
> 
> 

Am Ende dieses Tutorials führen Sie drei Windows-Konsolen-Apps aus:

* **SimulatedDevice**, eine geänderte Version der im Tutorial [Erste Schritte mit IoT Hub] erstellten App, die jede Sekunde Datenpunkt-D2C-Nachrichten und alle 10 Sekunden interaktive D2C-Nachrichten sendet. Für diese App wird das AMQPS-Protokoll für die Kommunikation mit IoT Hub verwendet.
* **ProcessDeviceToCloudMessages** verwendet die [EventProcessorHost] -Klasse zum Abrufen von Nachrichten aus dem Event Hubs-kompatiblen Endpunkt. Dann speichert sie zuverlässig Datenpunktnachrichten in Azure-Blobspeicher und leitet interaktive Nachrichten an eine Service Bus-Warteschlange weiter.
* **ProcessD2CInteractiveMessages** entfernt die interaktiven Nachrichten aus der Service Bus-Warteschlange.

> [!NOTE]
> IoT Hub verfügt über SDK-Unterstützung für zahlreiche Geräteplattformen und Sprachen, z.B. C, Java und JavaScript. Informationen zum Ersetzen des simulierten Geräts in diesem Tutorial durch ein physisches Gerät und zum Verbinden von Geräten mit einem IoT Hub finden Sie im [Azure IoT Developer Center].
> 
> 

Dieses Tutorial kann direkt auf andere Möglichkeiten zum Verarbeiten von Event Hubs-kompatiblen Nachrichten übertragen werden, z.B. Projekte vom Typ [HDInsight (Hadoop)]. Weitere Informationen finden Sie unter [Entwicklungsleitfaden für Azure IoT Hub – Device-to-Cloud].

Für dieses Lernprogramm benötigen Sie Folgendes:

* Microsoft Visual Studio 2015.
* Ein aktives Azure-Konto. <br/>Falls Sie über kein Azure-Abonnement verfügen, können Sie in wenigen Minuten ein [kostenloses Konto](https://azure.microsoft.com/free/) erstellen.

Sie sollten über grundlegende Kenntnisse zu [Azure Storage] und [Azure Service Bus] verfügen.

## <a name="send-interactive-messages-from-a-simulated-device"></a>Senden interaktiver Nachrichten von einem simulierten Gerät
In diesem Abschnitt ändern Sie die simulierte Geräteanwendung, die Sie im Tutorial [Erste Schritte mit IoT Hub] erstellt haben, sodass sie interaktive D2C-Nachrichten an IoT Hub sendet.

1. Fügen Sie in Visual Studio im **SimulatedDevice**-Projekt der **Program**-Klasse die folgende Methode hinzu.
   
    ```
    private static async void SendDeviceToCloudInteractiveMessagesAsync()
    {
      while (true)
      {
        var interactiveMessageString = "Alert message!";
        var interactiveMessage = new Message(Encoding.ASCII.GetBytes(interactiveMessageString));
        interactiveMessage.Properties["messageType"] = "interactive";
        interactiveMessage.MessageId = Guid.NewGuid().ToString();
   
        await deviceClient.SendEventAsync(interactiveMessage);
        Console.WriteLine("{0} > Sending interactive message: {1}", DateTime.Now, interactiveMessageString);
   
        Task.Delay(10000).Wait();
      }
    }
    ```
   
    Diese Methode ähnelt der **SendDeviceToCloudMessagesAsync**-Methode im Projekt **SimulatedDevice**. Der einzige Unterschied besteht darin, dass Sie jetzt die Systemeigenschaft **MessageId** und die Benutzereigenschaft **messageType** festlegen.
    Der Code weist der **MessageId**-Eigenschaft einen global eindeutigen Bezeichner (GUID) zu. Der Service Bus kann mit diesem Bezeichner Nachrichten, die er empfängt, deduplizieren. Im Beispiel wird die **messageType**-Eigenschaft verwendet, um interaktive Nachrichten von Datenpunktnachrichten zu unterscheiden. Die Anwendung übergibt diese Informationen in Nachrichteneigenschaften anstatt im Nachrichtentext. Auf diese Weise muss der Ereignisprozessor zum Nachrichtenrouting nicht die gesamte Nachricht deserialisieren.
   
   > [!NOTE]
   > Es ist wichtig, die **MessageId** zur Deduplizierung interaktiver Nachrichten im Gerätecode zu erstellen. Zeitweise Netzwerkkommunikation oder andere Fehler könnten dazu führen, dass mehrere Neuübertragungen der gleichen Nachricht von diesem Gerät erfolgen. Sie können anstelle einer GUID auch eine semantische Nachrichten-ID verwenden, z.B. einen Hash der relevanten Nachrichtendatenfelder.
   > 
   > 
2. Fügen Sie in der **Main**-Methode unmittelbar vor der Zeile `Console.ReadLine()` die folgende Methode hinzu:
   
    ````
    SendDeviceToCloudInteractiveMessagesAsync();
    ````
   
   > [!NOTE]
   > Der Einfachheit halber wird in diesem Tutorial keine Wiederholungsrichtlinie implementiert. Im Produktionscode sollten Sie eine Wiederholungsrichtlinie implementieren (etwa einen exponentiellen Backoff), wie im MSDN-Artikel zum [Behandeln vorübergehender Fehler]beschrieben.
   > 
   > 

## <a name="process-device-to-cloud-messages"></a>Verarbeiten von Gerät-zu-Cloud-Nachrichten
In diesem Abschnitt erstellen Sie eine Windows-Konsolen-App, die D2C-Nachrichten von IoT Hub verarbeitet. IoT Hub macht einen mit [Event Hubs]kompatiblen Endpunkt verfügbar, der einer Anwendung das Lesen von D2C-Nachrichten ermöglicht. In diesem Lernprogramm wird die [EventProcessorHost] -Klasse zum Verarbeiten dieser Nachrichten in einer Konsolen-App verwendet. Weitere Informationen zum Verarbeiten von Nachrichten von Event Hubs finden Sie im Tutorial [Erste Schritte mit Event Hubs] .

Die Herausforderung beim Implementieren der zuverlässigen Speicherung von Datenpunktnachrichten oder der Weiterleitung von interaktiven Nachrichten besteht darin, dass die Ereignisverarbeitung davon abhängt, dass der Nachrichtenconsumer Prüfpunkte für den Status bereitstellt. Darüber hinaus sollten Sie beim Lesen von Event Hubs Prüfpunkte für große Batches festlegen, um einen hohen Durchsatz zu erzielen. Hieraus ergibt sich die Möglichkeit einer doppelten Verarbeitung für eine große Zahl von Nachrichten, wenn ein Fehler vorliegt und Sie zum vorherigen Prüfpunkt zurückkehren. In diesem Tutorial lernen Sie, wie Azure-Speicherschreibvorgänge und Service Bus-Deduplizierungsfenster mit **EventProcessorHost** -Prüfpunkten synchronisiert werden.

Um Nachrichten zuverlässig in den Azure-Speicher zu schreiben, wird im Beispiel das Feature für den individuellen Block-Commit von [Blockblobs][Azure-Blockblobs] verwendet. Der Ereignisprozessor sammelt Nachrichten im Arbeitsspeicher, bis der Zeitpunkt gekommen ist, einen Prüfpunkt bereitzustellen. Dies ist z.B. der Fall, wenn der Puffer, der die gesammelten Nachrichten enthält, die maximale Blockgröße von 4MB erreicht, oder nach dem Verstreichen des Zeitfensters zur Service Bus-Deduplizierung. Vor dem Prüfpunkt führt der Code für den Blob dann einen Commit für einen neuen Block durch.

Der Ereignisprozessor nutzt Event Hubs-Nachrichten-Offsets als Block-IDs. Dieser Mechanismus ermöglicht dem Ereignisprozessor eine Überprüfung der Deduplizierung vor dem Ausführen eines Commits für den neuen Block an den Speicher, und ein etwaiger Konflikt zwischen dem Commit eines Blocks und dem Prüfpunkt wird verhindert.

> [!NOTE]
> In diesem Tutorial wird ein einzelnes Speicherkonto zum Schreiben aller Nachrichten verwendet, die von IoT Hub abgerufen werden. Informationen dazu, ob Sie in Ihrer Lösung mehrere Azure Storage-Konten benötigen, finden Sie in den [Richtlinien zur Azure Storage-Skalierbarkeit].
> 
> 

Die Anwendung nutzt die Service Bus-Deduplizierungsfunktion, um beim Verarbeiten interaktiver Nachrichten Duplikate zu vermeiden. Das simulierte Gerät versieht jede interaktive Nachricht mit einer eindeutigen **MessageId**. Mit diesen IDs kann Service Bus sicherstellen, dass im angegebenen Zeitfenster für die Deduplizierung nicht zwei Nachrichten mit der gleichen **MessageId** an die Empfänger übertragen werden. Diese Deduplizierung sorgt zusammen mit der Abschlusssemantik pro Nachricht von Service Bus-Warteschlangen dafür, dass die zuverlässige Verarbeitung von interaktiven Nachrichten vereinfacht wird.

Um sicherzustellen, dass Nachrichten nicht außerhalb des Deduplizierungsfensters erneut übermittelt werden, wird der **EventProcessorHost** -Prüfpunktmechanismus im Code mit dem Deduplizierungsfenster der Service Bus-Warteschlange synchronisiert. Für diese Synchronisierung wird mindestens einmal bei jedem Ablauf des Deduplizierungsfensters (in diesem Tutorial nach einer Stunde) ein Prüfpunkt erzwungen.

> [!NOTE]
> In diesem Tutorial wird eine einzelne partitionierte Service Bus-Warteschlange verwendet, um alle interaktiven Nachrichten zu verarbeiten, die von IoT Hub abgerufen werden. Weitere Informationen zur Erfüllung der Skalierbarkeitsanforderungen Ihrer Lösung mithilfe von Service Bus-Warteschlangen finden Sie in der [Dokumentation zu Azure Service Bus] .
> 
> 

### <a name="provision-an-azure-storage-account-and-a-service-bus-queue"></a>Bereitstellen eines Azure-Speicherkontos und einer Service Bus-Warteschlange
Zum Verwenden der [EventProcessorHost] -Klasse müssen Sie über ein Azure Storage-Konto verfügen, damit **EventProcessorHost** Prüfpunktinformationen erfassen kann. Sie können ein vorhandenes Speicherkonto verwenden oder mithilfe der Anweisungen unter [Informationen zu Azure Storage] ein neues Konto erstellen. Notieren Sie sich die Verbindungszeichenfolge für das Speicherkonto.

> [!NOTE]
> Stellen Sie beim Kopieren und Einfügen der Speicherkonto-Verbindungszeichenfolge sicher, dass keine Leerzeichen aufgenommen werden.
> 
> 

Sie benötigen außerdem eine Service Bus-Warteschlange, um die zuverlässige Verarbeitung von interaktiven Nachrichten zu aktivieren. Sie können eine Warteschlange programmgesteuert mit einem Deduplizierungsfenster von einer Stunde erstellen. Dies ist unter [Erste Schritte mit Service Bus-Warteschlangen][Service Bus-Warteschlange] beschrieben. Sie können aber auch im [klassischen Azure-Portal][lnk-classic-portal] die folgenden Schritte ausführen:

1. Klicken Sie in der Ecke unten links auf **Neu** . Klicken Sie dann auf **App Services** > **Service Bus** > **Warteschlange** > **Benutzerdefiniert erstellen**. Geben Sie den Namen **d2ctutorial**ein, wählen Sie eine Region aus, und verwenden Sie einen vorhandenen Namespace, oder erstellen Sie einen neuen Namespace. Wählen Sie auf der nächsten Seite **Doppelte Erkennung aktivieren** aus, und legen Sie für **Fenster für Duplikaterkennungsverlauf-Zeitpunkt** eine Stunde fest. Klicken Sie dann auf das Kontrollkästchen in der rechten unteren Ecke, um die Warteschlangenkonfiguration zu speichern.
   
    ![Erstellen einer Warteschlange im Azure-Portal][30]
2. Klicken Sie in der Liste mit den Service Bus-Warteschlangen auf **d2ctutorial** und dann auf **Konfigurieren**. Erstellen Sie zwei SAS-Richtlinien: eine mit dem Namen **send** und Berechtigungen zum **Senden** und eine mit dem Namen **listen** und Berechtigungen zum **Lauschen**. Wenn Sie fertig sind, klicken Sie unten auf **Speichern** .
   
    ![Konfigurieren einer Warteschlange im Azure-Portal][31]
3. Klicken Sie oben auf **Dashboard** und am unteren Rand auf **Verbindungsinformationen**. Notieren Sie sich die beiden Verbindungszeichenfolgen.
   
    ![Warteschlangen-Dashboards im Azure-Portal][32]

### <a name="create-the-event-processor"></a>Erstellen des Ereignisprozessors
1. Klicken Sie in der aktuellen Visual Studio-Projektmappe auf **Datei** > **Hinzufügen** > **Neues Projekt**, um mit der Projektvorlage **Konsolenanwendung** ein Visual C#-Windows-Projekt zu erstellen. Stellen Sie sicher, dass .NET-Framework-Version 4.5.1 oder höher verwendet wird. Geben Sie dem Projekt den Namen **ProcessDeviceToCloudMessages**, und klicken Sie auf **OK**.
   
    ![Neues Projekt in Visual Studio][10]
2. Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf das Projekt **ProcessDeviceToCloudMessages**, und klicken Sie dann auf **NuGet-Pakete verwalten**. Das Dialogfeld **NuGet-Paket-Manager** wird angezeigt.
3. Suchen Sie nach **WindowsAzure.ServiceBus**, klicken Sie auf **Installieren**, und akzeptieren Sie die Nutzungsbedingungen. Mit diesem Vorgang wird das NuGet-Paket [Azure Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus) mit allen Abhängigkeiten heruntergeladen und installiert, und dem Projekt wird ein Verweis auf das Paket hinzugefügt.
4. Suchen Sie nach **Microsoft.Azure.ServiceBus.EventProcessorHost**, klicken Sie auf **Installieren**, und akzeptieren Sie die Nutzungsbedingungen. Mit diesem Vorgang wird das NuGet-Paket [Azure Service Bus Event Hub - EventProcessorHost](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost) mit allen Abhängigkeiten heruntergeladen und installiert, und dem Projekt wird ein Verweis auf das Paket hinzugefügt.
5. Klicken Sie mit der rechten Maustaste auf das Projekt **ProcessDeviceToCloudMessages**, klicken Sie auf **Hinzufügen** und dann auf **Klasse**. Geben Sie der neuen Klasse den Namen **StoreEventProcessor**, und klicken Sie auf **OK**, um die Klasse zu erstellen.
6. Fügen Sie die folgenden Anweisungen am Anfang der Datei "StoreEventProcessor.cs" hinzu:
   
    ```
    using System.IO;
    using System.Diagnostics;
    using System.Security.Cryptography;
    using Microsoft.ServiceBus.Messaging;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```
7. Ersetzen Sie den Text der Klasse durch folgenden Code:
   
    ```
    class StoreEventProcessor : IEventProcessor
    {
      private const int MAX_BLOCK_SIZE = 4 * 1024 * 1024;
      public static string StorageConnectionString;
      public static string ServiceBusConnectionString;
   
      private CloudBlobClient blobClient;
      private CloudBlobContainer blobContainer;
      private QueueClient queueClient;
   
      private long currentBlockInitOffset;
      private MemoryStream toAppend = new MemoryStream(MAX_BLOCK_SIZE);
   
      private Stopwatch stopwatch;
      private TimeSpan MAX_CHECKPOINT_TIME = TimeSpan.FromHours(1);
   
      public StoreEventProcessor()
      {
        var storageAccount = CloudStorageAccount.Parse(StorageConnectionString);
        blobClient = storageAccount.CreateCloudBlobClient();
        blobContainer = blobClient.GetContainerReference("d2ctutorial");
        blobContainer.CreateIfNotExists();
        queueClient = QueueClient.CreateFromConnectionString(ServiceBusConnectionString);
      }
   
      Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
      {
        Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
        return Task.FromResult<object>(null);
      }
   
      Task IEventProcessor.OpenAsync(PartitionContext context)
      {
        Console.WriteLine("StoreEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);
   
        if (!long.TryParse(context.Lease.Offset, out currentBlockInitOffset))
        {
          currentBlockInitOffset = 0;
        }
        stopwatch = new Stopwatch();
        stopwatch.Start();
   
        return Task.FromResult<object>(null);
      }
   
      async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
      {
        foreach (EventData eventData in messages)
        {
          byte[] data = eventData.GetBytes();
   
          if (eventData.Properties.ContainsKey("messageType") && (string) eventData.Properties["messageType"] == "interactive")
          {
            var messageId = (string) eventData.SystemProperties["message-id"];
   
            var queueMessage = new BrokeredMessage(new MemoryStream(data));
            queueMessage.MessageId = messageId;
            queueMessage.Properties["messageType"] = "interactive";
            await queueClient.SendAsync(queueMessage);
   
            WriteHighlightedMessage(string.Format("Received interactive message: {0}", messageId));
            continue;
          }
   
          if (toAppend.Length + data.Length > MAX_BLOCK_SIZE || stopwatch.Elapsed > MAX_CHECKPOINT_TIME)
          {
            await AppendAndCheckpoint(context);
          }
          await toAppend.WriteAsync(data, 0, data.Length);
   
          Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
            context.Lease.PartitionId, Encoding.UTF8.GetString(data)));
        }
      }
   
      private async Task AppendAndCheckpoint(PartitionContext context)
      {
        var blockIdString = String.Format("startSeq:{0}", currentBlockInitOffset.ToString("0000000000000000000000000"));
        var blockId = Convert.ToBase64String(ASCIIEncoding.ASCII.GetBytes(blockIdString));
        toAppend.Seek(0, SeekOrigin.Begin);
        byte[] md5 = MD5.Create().ComputeHash(toAppend);
        toAppend.Seek(0, SeekOrigin.Begin);
   
        var blobName = String.Format("iothubd2c_{0}", context.Lease.PartitionId);
        var currentBlob = blobContainer.GetBlockBlobReference(blobName);
   
        if (await currentBlob.ExistsAsync())
        {
          await currentBlob.PutBlockAsync(blockId, toAppend, Convert.ToBase64String(md5));
          var blockList = await currentBlob.DownloadBlockListAsync();
          var newBlockList = new List<string>(blockList.Select(b => b.Name));
   
          if (newBlockList.Count() > 0 && newBlockList.Last() != blockId)
          {
            newBlockList.Add(blockId);
            WriteHighlightedMessage(String.Format("Appending block id: {0} to blob: {1}", blockIdString, currentBlob.Name));
          }
          else
          {
            WriteHighlightedMessage(String.Format("Overwriting block id: {0}", blockIdString));
          }
          await currentBlob.PutBlockListAsync(newBlockList);
        }
        else
        {
          await currentBlob.PutBlockAsync(blockId, toAppend, Convert.ToBase64String(md5));
          var newBlockList = new List<string>();
          newBlockList.Add(blockId);
          await currentBlob.PutBlockListAsync(newBlockList);
   
          WriteHighlightedMessage(String.Format("Created new blob", currentBlob.Name));
        }
   
        toAppend.Dispose();
        toAppend = new MemoryStream(MAX_BLOCK_SIZE);
   
        // checkpoint.
        await context.CheckpointAsync();
        WriteHighlightedMessage(String.Format("Checkpointed partition: {0}", context.Lease.PartitionId));
   
        currentBlockInitOffset = long.Parse(context.Lease.Offset);
        stopwatch.Restart();
      }
   
      private void WriteHighlightedMessage(string message)
      {
        Console.ForegroundColor = ConsoleColor.Yellow;
        Console.WriteLine(message);
        Console.ResetColor();
      }
    }
    ```
   
    Die **EventProcessorHost** -Klasse ruft diese Klasse auf, um von IoT Hub empfangene D2C-Nachrichten zu verarbeiten. Mit dem Code in dieser Klasse wird die Logik zum zuverlässigen Speichern von Nachrichten in einem Blobcontainer implementiert, und interaktive Nachrichten werden an die Service Bus-Warteschlange weitergeleitet.
   
    Die **OpenAsync**-Methode initialisiert die **currentBlockInitOffset**-Variable. Diese dient zum Nachverfolgen des aktuellen Offsets der ersten Nachricht, die von diesem Ereignisprozessor gelesen wird. Denken Sie daran, dass jeder Prozessor für eine bestimmte Partition verantwortlich ist.
   
    Mit der **ProcessEventsAsync**-Methode wird ein Nachrichtenbatch von IoT Hub empfangen und wie folgt verarbeitet: Interaktive Nachrichten werden an die Service Bus-Warteschlange gesendet, und Datenpunktnachrichten werden an den Arbeitsspeicherpuffer **toAppend** angefügt. Falls der Arbeitsspeicherpuffer das Limit von 4MB erreicht oder die Deduplizierungszeitfenster abgelaufen sind (in diesem Tutorial eine Stunde nach einem Prüfpunkt), löst die Anwendung einen Prüfpunkt aus.
   
    Mit der **AppendAndCheckpoint**-Methode wird zuerst eine Block-ID für den anzufügenden Block generiert. Für Azure Storage müssen alle Block-IDs die gleiche Länge besitzen. Die Methode füllt den Offset daher mit führenden Nullen auf: `currentBlockInitOffset.ToString("0000000000000000000000000")`. Wenn ein Block mit dieser ID im Blob bereits vorhanden ist, überschreibt die Methode ihn mit dem aktuellen Inhalt des Puffers.
   
   > [!NOTE]
   > Um den Code zu vereinfachen, wird in diesem Tutorial eine einzelne Blob-Datei pro Partition zum Speichern von Nachrichten verwendet. Für eine echte Lösung würde das „File Rolling“ implementiert werden, indem zusätzliche Dateien erstellt werden, wenn ein bestimmter Zeitraum verstrichen ist oder wenn sie eine bestimmte Größe erreichen. Denken Sie daran, dass ein Azure-Blockblob höchstens 195GB Daten enthalten kann.
   > 
   > 
8. Fügen Sie am Anfang der **Program**-Klasse die folgende **using**-Anweisung hinzu:
   
    ```
    using Microsoft.ServiceBus.Messaging;
    ```
9. Ändern Sie die **Main**-Methode in der **Program**-Klasse wie folgt. Ersetzen Sie **{iot hub connection string}** durch die **iothubowner**-Verbindungszeichenfolge aus dem Tutorial [Erste Schritte mit IoT Hub]. Ersetzen Sie die Speicherverbindungszeichenfolge durch die Verbindungszeichenfolge, die Sie zu Beginn dieses Abschnitts notiert haben. Ersetzen Sie die Service Bus-Verbindungszeichenfolge durch die **Senden**-Berechtigungen für die Warteschlange **d2ctutorial**, die Sie zu Beginn dieses Abschnitts notiert haben:
   
    ```
    static void Main(string[] args)
    {
      string iotHubConnectionString = "{iot hub connection string}";
      string iotHubD2cEndpoint = "messages/events";
      StoreEventProcessor.StorageConnectionString = "{storage connection string}";
      StoreEventProcessor.ServiceBusConnectionString = "{service bus send connection string}";
   
      string eventProcessorHostName = Guid.NewGuid().ToString();
      EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, iotHubD2cEndpoint, EventHubConsumerGroup.DefaultGroupName, iotHubConnectionString, StoreEventProcessor.StorageConnectionString, "messages-events");
      Console.WriteLine("Registering EventProcessor...");
      eventProcessorHost.RegisterEventProcessorAsync<StoreEventProcessor>().Wait();
   
      Console.WriteLine("Receiving. Press enter key to stop worker.");
      Console.ReadLine();
      eventProcessorHost.UnregisterEventProcessorAsync().Wait();
    }
    ```
   
   > [!NOTE]
   > Der Einfachheit halber wird in diesem Tutorial eine einzelne Instanz der [EventProcessorHost] -Klasse verwendet. Weitere Informationen finden Sie im [Programmierleitfaden für Event Hubs].
   > 
   > 

## <a name="receive-interactive-messages"></a>Empfangen von interaktiven Nachrichten
In diesem Abschnitt schreiben Sie eine Windows-Konsolen-App, die interaktive Nachrichten aus der Service Bus-Warteschlange empfängt. Weitere Informationen zum Erstellen einer Lösung mit Service Bus finden Sie unter [.NET-Anwendungen mit mehreren Ebenen unter Verwendung von Azure Service Bus-Warteschlangen][].

1. Erstellen Sie in der aktuellen Visual Studio-Projektmappe mithilfe der Projektvorlage **Konsolenanwendung** ein Visual C#-Windows-Projekt. Geben Sie dem Projekt den Namen **ProcessD2CInteractiveMessages**.
2. Klicken Sie im Projektmappen-Explorer mit der rechten Maustaste auf das Projekt **ProcessD2CInteractiveMessages**, und klicken Sie dann auf **NuGet-Pakete verwalten**. Mit diesem Vorgang wird das Fenster **NuGet-Paket-Manager** angezeigt.
3. Suchen Sie nach **WindowsAzure.ServiceBus**, klicken Sie auf **Installieren**, und akzeptieren Sie die Nutzungsbedingungen. Mit diesem Vorgang wird [Azure Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus)mit allen Abhängigkeiten heruntergeladen, installiert und mit einem Verweis versehen.
4. Fügen Sie am Anfang der Datei **Program.cs** die folgenden **using**-Anweisungen hinzu:
   
    ```
    using System.IO;
    using Microsoft.ServiceBus.Messaging;
    ```
5. Fügen Sie abschließend der **Main** -Methode die folgenden Zeilen hinzu. Ersetzen Sie die Verbindungszeichenfolge durch **Lauschen**-Berechtigungen für die Warteschlange **d2ctutorial**:
   
    ```
    Console.WriteLine("Process D2C Interactive Messages app\n");
   
    string connectionString = "{service bus listen connection string}";
    QueueClient Client = QueueClient.CreateFromConnectionString(connectionString);
   
    OnMessageOptions options = new OnMessageOptions();
    options.AutoComplete = false;
    options.AutoRenewTimeout = TimeSpan.FromMinutes(1);
   
    Client.OnMessage((message) =>
    {
      try
      {
        var bodyStream = message.GetBody<Stream>();
        bodyStream.Position = 0;
        var bodyAsString = new StreamReader(bodyStream, Encoding.ASCII).ReadToEnd();
   
        Console.WriteLine("Received message: {0} messageId: {1}", bodyAsString, message.MessageId);
   
        message.Complete();
      }
      catch (Exception)
      {
        message.Abandon();
      }
    }, options);
   
    Console.WriteLine("Receiving interactive messages from SB queue...");
    Console.WriteLine("Press any key to exit.");
    Console.ReadLine();
    ```

## <a name="run-the-applications"></a>Ausführen der Anwendungen
Sie können jetzt die Anwendung ausführen.

1. Klicken Sie in Visual Studio im Projektmappen-Explorer mit der rechten Maustaste auf Ihre Projektmappe, und wählen Sie **Startprojekte festlegen**aus. Wählen Sie **Mehrere Startprojekte** aus, und wählen Sie dann für die Projekte **ProcessDeviceToCloudMessages**, **SimulatedDevice** und **ProcessD2CInteractiveMessages** die Aktion **Starten** aus.
2. Drücken Sie **F5**, um die drei Konsolenanwendungen zu starten. Die Anwendung **ProcessD2CInteractiveMessages** sollte jede interaktive Nachricht verarbeiten, die von der Anwendung **SimulatedDevice** gesendet wird.
   
   ![Drei Konsolenanwendungen][50]

> [!NOTE]
> Um Aktualisierungen in Ihrer Blobdatei sehen zu können, müssen Sie unter Umständen für die Konstante **MAX_BLOCK_SIZE** in der **StoreEventProcessor**-Klasse einen niedrigeren Wert festlegen (beispielsweise **1024**). Diese Änderung ist nützlich, weil es bei den mit dem simulierten Gerät gesendeten Daten einige Zeit dauert, bis die Blockgröße erreicht wird. Bei einer kleineren Blockgröße müssen Sie nicht so lange warten, bis das Blob erstellt und aktualisiert wird. Allerdings sorgt die Verwendung einer höheren Blockgröße für eine bessere Skalierbarkeit der Anwendung.
> 
> 

## <a name="next-steps"></a>Nächste Schritte
In diesem Tutorial haben Sie gelernt, wie Sie mit der [EventProcessorHost] -Klasse zuverlässig Datenpunktnachrichten und interaktive D2C-Nachrichten verarbeiten.

Im Tutorial [Gewusst wie: Senden von Cloud-to-Device-Nachrichten mit IoT Hub][lnk-c2d] erfahren Sie, wie Sie Nachrichten von Ihrem Back-End an Ihre Geräte senden.

Beispiele vollständiger End-to-End-Lösungen, die IoT Hub nutzen, finden Sie unter [Azure IoT Suite][lnk-suite].

Weitere Informationen zum Entwickeln von Lösungen mit IoT Hub finden Sie im [IoT Hub-Entwicklerhandbuch].

<!-- Images. -->
[50]: ./media/iot-hub-csharp-csharp-process-d2c/run1.png
[10]: ./media/iot-hub-csharp-csharp-process-d2c/create-identity-csharp1.png

[30]: ./media/iot-hub-csharp-csharp-process-d2c/createqueue2.png
[31]: ./media/iot-hub-csharp-csharp-process-d2c/createqueue3.png
[32]: ./media/iot-hub-csharp-csharp-process-d2c/createqueue4.png

<!-- Links -->

[Azure Blob Storage]: ../storage/storage-dotnet-how-to-use-blobs.md
[Azure Data Factory]: https://azure.microsoft.com/documentation/services/data-factory/
[HDInsight (Hadoop)]: https://azure.microsoft.com/documentation/services/hdinsight/
[Service Bus-Warteschlange]: ../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md

[Entwicklerhandbuch für Azure IoT Hub – Device-to-Cloud]: iot-hub-devguide-messaging.md

[Azure Storage]: https://azure.microsoft.com/documentation/services/storage/
[Azure Service Bus]: https://azure.microsoft.com/documentation/services/service-bus/

[IoT Hub-Entwicklerhandbuch]: iot-hub-devguide.md
[Erste Schritte mit IoT Hub]: iot-hub-csharp-csharp-getstarted.md
[Azure IoT Developer Center]: https://azure.microsoft.com/develop/iot
[lnk-service-fabric]: https://azure.microsoft.com/documentation/services/service-fabric/
[lnk-stream-analytics]: https://azure.microsoft.com/documentation/services/stream-analytics/
[lnk-event-hubs]: https://azure.microsoft.com/documentation/services/event-hubs/
[Behandeln vorübergehender Fehler]: https://msdn.microsoft.com/library/hh675232.aspx

<!-- Links -->
[Informationen zu Azure Storage]: ../storage/storage-create-storage-account.md#create-a-storage-account
[Erste Schritte mit Event Hubs]: ../event-hubs/event-hubs-csharp-ephcs-getstarted.md
[Richtlinien zur Azure Storage-Skalierbarkeit]: ../storage/storage-scalability-targets.md
[Azure-Blockblobs]: https://msdn.microsoft.com/library/azure/ee691964.aspx
[Event Hubs]: ../event-hubs/event-hubs-overview.md
[EventProcessorHost]: http://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.eventprocessorhost(v=azure.95).aspx
[Programmierleitfaden für Event Hubs]: ../event-hubs/event-hubs-programming-guide.md
[Behandeln vorübergehender Fehler]: https://msdn.microsoft.com/library/hh680901(v=pandp.50).aspx
[Erstellen von .NET-Anwendungen mit mehreren Ebenen unter Verwendung von Service Bus]: ../service-bus-messaging/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md

[lnk-classic-portal]: https://manage.windowsazure.com
[lnk-c2d]: iot-hub-csharp-csharp-process-d2c.md
[lnk-suite]: https://azure.microsoft.com/documentation/suites/iot-suite/



<!--HONumber=Oct16_HO2-->


