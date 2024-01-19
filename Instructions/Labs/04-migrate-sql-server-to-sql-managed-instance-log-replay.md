---
lab:
  title: Migrieren von SQL Server-Datenbanken zu Azure SQL Managed Instance mithilfe des Protokollwiedergabediensts
---

# Migrieren von SQL Server-Datenbanken zu Azure SQL Managed Instance mithilfe des Protokollwiedergabediensts

In dieser Übung erfahren Sie, wie Sie eine SQL Server-Datenbank mithilfe des Protokollwiedergabediensts zu Azure SQL Managed Instance migrieren. 

Sie beginnen mit der Bereitstellung einer Azure SQL Managed Instance. Dann verwenden Sie den Protokollwiedergabedienst, um eine Onlinemigration einer SQL Server-Datenbank zu Azure SQL Managed Instance durchzuführen. Zudem erfahren Sie, wie Sie den Migrationsprozess in PowerShell überwachen.

Diese Übung dauert ca. **45** Minuten.

> **Hinweis**: Um diese Übung abzuschließen, benötigen Sie Zugriff auf ein Azure-Abonnement, um Azure-Ressourcen zu erstellen. Wenn Sie kein Azure-Abonnement besitzen, können Sie ein [kostenloses Konto](https://azure.microsoft.com/free/?azure-portal=true) erstellen, bevor Sie beginnen.

## Vorbereitung

Um diese Übung auszuführen, benötigen Sie Folgendes:

| Element | Beschreibung |
| --- | --- |
| **Zielserver** | Eine Azure SQL Managed Instance Wir erstellen ihn während dieser Übung.|
| **Quellserver** | Eine Instanz von SQL Server 2019 oder eine [neuere Version](https://www.microsoft.com/en-us/sql-server/sql-server-downloads), die auf einem Server Ihrer Einstellung installiert ist. |
| **Quelldatenbank** | Die einfache [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure)-Datenbank, die auf der SQL Server-Instanz wiederhergestellt werden soll. |
| **Azure Data Studio** | Installieren Sie [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio) auf demselben Server, auf dem sich die Quelldatenbank befindet. Wenn sie bereits installiert ist, aktualisieren Sie es, um sicherzustellen, dass Sie die neueste Version verwenden. |

## Wiederherstellen einer SQL Server-Datenbank

Stellen wir die Datenbank *AdventureWorksLT* auf der SQL Server-Instanz wieder her. Diese Datenbank dient als Quelldatenbank für die Übung in diesem Lab. Sie können diese Schritte überspringen, wenn die Datenbank bereits wiederhergestellt wurde.

1. Wählen Sie die Windows-Startschaltfläche aus, und geben Sie „SSMS“ ein. Wählen Sie **Microsoft SQL Server Management Studio 18** aus der Liste aus.  

1. Beachten Sie, dass das Dialogfeld **Mit Server verbinden** beim Öffnen von SSMS vorab mit dem Standardinstanznamen ausgefüllt wird. Wählen Sie **Verbinden** aus.

1. Wählen Sie den Ordner **Datenbanken**, und dann **Neue Abfrage**.

1. Kopieren Sie den folgenden T-SQL-Code, und fügen Sie ihn in das neue Abfragefenster ein. Führen Sie die Abfrage aus, um die Datenbank wiederherzustellen.

    ```sql
    RESTORE DATABASE AdventureWorksLT
    FROM DISK = 'C:\LabFiles\AdventureWorksLT2019.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorksLT2019_Data' 
            TO 'C:\LabFiles\AdventureWorksLT2019.mdf',
          MOVE 'AdventureWorksLT2019_Log'
            TO 'C:\LabFiles\AdventureWorksLT2019.ldf';
    ```

    > **Hinweis**: Stellen Sie sicher, dass der Name der Datenbanksicherungsdatei und der Pfad im obigen Beispiel ihrer tatsächlichen Sicherungsdatei entsprechen. Ist dies nicht der Fall, kann der Befehl fehlschlagen.

1. Nach Abschluss der Wiederherstellung sollte eine erfolgreiche Meldung angezeigt werden.

## Bereitstellen einer Azure SQL Managed Instance

Führen Sie die folgenden Schritte aus, um eine Azure SQL Managed Instance zu erstellen:

1. Melden Sie sich beim [Azure-Portal](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true) an, und wählen Sie dann oben links **Ressource erstellen** aus.
1. Suchen Sie nach **Verwaltete Instanz**, wählen Sie **Azure SQL Managed Instance** und dann **Erstellen** aus.
1. Füllen Sie das Formular „SQL Managed Instance“ mit den Angaben aus der folgenden Tabelle aus:

    |  | Vorgeschlagener Wert |
    |---|---|
    | **Abonnement** | Ihr Abonnement |
    | **Name der verwalteten Instanz** | Ein gültiger Name |
    | **Administratoranmeldung für verwaltete Instanz** | Jeder gültige Benutzername Verwenden Sie nicht „serveradmin“. Hierbei handelt es sich um eine reservierte Rolle auf Serverebene. |
    | **Kennwort** | Jedes Kennwort, das länger als 16 Zeichen ist und die Komplexitätsanforderungen erfüllt |
    | **Zeitzone** | Die Zeitzone, die von Ihrer verwalteten Instanz beachtet werden soll. |
    | **Sortierung** | Die Sortierung, die Sie für die verwaltete Instanz verwenden möchten Wenn Sie Datenbanken von SQL Server migrieren, überprüfen Sie mit „SELECT SERVERPROPERTY(N'Collation')“ die Quellsortierung, und verwenden Sie diesen Wert. |
    | **Location** | Die Azure-Region, in der Sie die verwaltete Instanz erstellen möchten |
    | **Virtuelles Netzwerk** | Klicken Sie auf „Neues virtuelles Netzwerk erstellen“, oder wählen Sie ein gültiges virtuelles Netzwerk und ein Subnetz aus. |
    | **Öffentlichen Endpunkt aktivieren** | Aktivieren Sie diese Option, um einen öffentlichen Endpunkt zu aktivieren, mit dem Clients außerhalb von Azure auf die Datenbank zugreifen können. |
    | **Zugriff erlauben von** | Zugriff über Azure-Dienste, das Internet oder kein Zugriff |
    | **Verbindungstyp** | Wählen Sie zwischen dem Proxy- und dem Umleitungsverbindungstyp. |
    | **Ressourcengruppe** | Eine neue oder vorhandene Ressourcengruppe. |

1. Wählen Sie **Tarif** aus, um die Größe der Compute- und Speicherressourcen festzulegen und die Tarifoptionen zu überprüfen. 
1. Wählen Sie anschließend **Übernehmen** aus, um die Auswahl zu speichern, und wählen Sie dann **Erstellen** aus, um die verwaltete SQL-Datenbank-Instanz bereitzustellen.
1. Wählen Sie das Symbol **Benachrichtigungen** aus, um den Status der Bereitstellung anzuzeigen.
1. Wählen Sie **Die Bereitstellung wird ausgeführt.**, um das Fenster für die verwaltete Instanz zu öffnen und den Bereitstellungsstatus weiter zu verfolgen.

## Erstellen eines Azure Blob Storage-Kontos und eines Containers

Erstellen Sie ein Azure Blob Storage-Konto in der gleichen Region wie Ihre Azure SQL Managed Instance. Hier werden Sie Ihre Datenbanksicherungen für die Migration speichern.

1. Wechseln Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich mit Ihren Kontoanmeldeinformationen an.
1. Wählen Sie im linken Menü **Alle Dienste** aus, und suchen Sie dann nach *„Speicherkonten“*. Wählen Sie **Speicherkonten** aus, um die Seite „Speicherkonten“ zu öffnen.
1. Wählen Sie auf der Seite „Speicherkonten“ die Option **+ Hinzufügen** aus, um ein neues Speicherkonto zu erstellen.
1. Wählen Sie auf der Registerkarte **Grundlagen** der Seite **Speicherkonto erstellen** und das Abonnement aus, das Sie für das Speicherkonto verwenden möchten. Wählen Sie dann die Ressourcengruppe aus, die Ihre Azure SQL Managed Instance enthält.
1. Geben Sie einen eindeutigen Namen für das Speicherkonto ein. 
    
    > **Hinweis:** Der Name muss zwischen 3 und 24 Zeichen lang sein und darf nur Kleinbuchstaben und Ziffern enthalten.

1. Wählen Sie den Speicherort (die Region) aus, an dem sich Ihre Azure SQL Managed Instance befindet.
1. Wählen Sie die Leistungsstufe für das Speicherkonto aus.
1. Wählen Sie als Kontoart für das Speicherkonto **BlobStorage** aus. 
1. Wählen Sie für das Speicherkonto die Replikationsoption **Lokal redundanter Speicher (LRS)** aus.
1. Überprüfen Sie, und wählen Sie **Überprüfen + erstellen** aus, um das Speicherkonto zu erstellen.
1. Nachdem das Speicherkonto erstellt worden ist, wechseln Sie zur Seite „Speicherkonto“, und wählen Sie im linken Menü die Option **Container** aus. Wählen Sie anschließend **+ Container** aus, um einen neuen Container zu erstellen. Geben Sie für den Container einen Namen ein, und wählen Sie die öffentliche Zugriffsebene aus. 
1. Klicken Sie auf die Schaltfläche **Erstellen**, um den Container zu erstellen.

Nach Abschluss dieser Schritte verfügen Sie über ein Azure Blob Storage-Konto in derselben Region wie Ihre Azure SQL Managed Instance und einen Container, in dem Sie Ihre Datenbanksicherungen für die Migration speichern können.

## Sichern einer SQL Server-Datenbank

Erstellen wir nun eine vollständige Sicherung der Datenbank *AdventureWorksLT* in der SQL Server-Instanz, gefolgt von einer differenziellen Sicherung und Protokollsicherungen mit aktivierter Option `CHECKSUM`. 

1. Wählen Sie die Windows-Startschaltfläche aus, und geben Sie „SSMS“ ein. Wählen Sie **Microsoft SQL Server Management Studio 18** aus der Liste aus.  
1. Beachten Sie, dass das Dialogfeld **Mit Server verbinden** beim Öffnen von SSMS vorab mit dem Standardinstanznamen ausgefüllt worden ist. Wählen Sie **Verbinden** aus.
1. Wählen Sie den Ordner **Datenbanken**, und dann **Neue Abfrage**.
1. Kopieren Sie im neuen Abfragefenster die folgende T-SQL und fügen Sie sie ein. Führen Sie die Abfrage aus, um die Datenbank wiederherzustellen.

    ```sql
    BACKUP DATABASE AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_full.bak'
    WITH CHECKSUM;

    BACKUP DATABASE AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_diff.dif'
    WITH DIFFERENTIAL, CHECKSUM;

    BACKUP LOG AdventureWorksLT
    TO DISK = 'C:\LabFiles\AdventureWorksLT_log.trn'
    WITH CHECKSUM;
    ```

    > **Hinweis:** Stellen Sie sicher, dass der Dateipfad im obigen Beispiel mit dem tatsächlichen Dateipfad übereinstimmt. Ist dies nicht der Fall, kann der Befehl fehlschlagen.

1. Nach Abschluss der Wiederherstellung sollte eine erfolgreiche Meldung angezeigt werden.
1. Wenn Sie eine Version von SQL Server ausführen (ab SQL Server 2012 SP1 CU2 und SQL Server 2014), können Sie Sicherungen von SQL Server direkt in Ihr Blob Storage-Konto aufnehmen, indem Sie die systemeigene SQL Server-Option `BACKUP TO URL` verwenden. 

    ```sql
    CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<containername>] 
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
    SECRET = '<SAS_TOKEN>';  
    GO
    
    -- Take a full database backup to a URL
    BACKUP DATABASE [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_full.bak'
    WITH INIT, COMPRESSION, CHECKSUM
    GO
    
    -- Take a differential database backup to a URL
    BACKUP DATABASE [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_diff.bak'  
    WITH DIFFERENTIAL, COMPRESSION, CHECKSUM
    GO
    
    -- Take a transactional log backup to a URL
    BACKUP LOG [AdventureWorksLT]
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<databasefolder>/AdventureWorksLT_log.trn'  
    WITH COMPRESSION, CHECKSUM
    ```

    > **Hinweis:** Wenn Sie sich für die Verwendung dieser Option entscheiden, können Sie den nächsten Abschnitt **Kopieren von Sicherungsdateien in das Azure Storage-Konto** überspringen.

## Kopieren von Sicherungsdateien in das Azure Storage-Konto

Jetzt kopieren wir die Sicherungsdateien in das Azure Blob Storage-Konto, das Sie zuvor erstellt haben.

1. Wechseln Sie zum [Azure-Portal](https://portal.azure.com), und melden Sie sich mit Ihren Kontoanmeldeinformationen an.
1. Wählen Sie im linken Menü **Speicherkonten** und anschließend das Speicherkonto aus, das Sie zuvor erstellt haben.
1. Scrollen Sie in der Übersichtsseite für das Speicherkonto nach unten zum Abschnitt **Blob-Dienst**, und wählen Sie **Container** aus. Wählen Sie den zuvor erstellten Container aus.
1. Klicken Sie am oberen Rand der Containerseite auf **Hochladen**. Wählen Sie auf der Seite **Blob hochladen** die Option **Ordner** aus, um den Ordner auszuwählen, der die Sicherungsdateien enthält, oder wählen Sie **Dateien** aus, um einzelne Sicherungsdateien auszuwählen. Sobald Sie die Dateien ausgewählt haben, wählen Sie **Hochladen** aus, um den Uploadvorgang zu starten.

## Überprüfen des Zugriffs

Sie müssen unbedingt überprüfen, ob Ihr SQL Server und Ihre SQL Managed Instance erfolgreich auf Ihr Blob Storage-Konto zugreifen können. Führen Sie zu diesem Zweck eine Beispieltestabfrage aus, um zu ermitteln, ob Ihre verwaltete Instanz auf die Sicherung im Container zugreifen kann.

1. Stellen Sie über SSMS eine Verbindung mit Ihrer SQL Managed Instance her.
1. Öffnen Sie den Abfrage-Editor, und führen Sie den Befehl aus.

```sql
CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/databases] 
WITH IDENTITY = 'SHARED ACCESS SIGNATURE' 
, SECRET = '<sastoken>' 

RESTORE HEADERONLY 
FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<containername>/<backup_file_name>.bak'
```
1. Wiederholen Sie diesen Prozess in Verbindung mit Ihrer SQL Server-Instanz aus.

## Verwenden des Protokollwiederherstellungsdiensts zum Wiederherstellen von Sicherungsdateien

Sie verwenden den Protokollwiederherstellungsdienst (Log Replay Service, LRS), um die Sicherungsdateien von Azure Blob Storage in Ihrer Azure SQL Managed Instance wiederherzustellen. LRS ist ein kostenloser Dienst, der auf der Protokollversandtechnologie von SQL Server basiert.

1. Scrollen Sie in der Übersichtsseite für das Speicherkonto nach unten zum Abschnitt **Blob-Dienst**, und wählen Sie **Container** aus. Wählen Sie den Container aus, in dem die Sicherungsdateien gespeichert worden sind.
1. Wählen Sie am oberen Rand der Containerseite **SAS generieren** aus. Wählen Sie auf der Seite **SAS generieren** die Berechtigungen aus, die Sie erteilen möchten, legen Sie die Start- und Ablaufzeit für das SAS-Token fest, und wählen Sie dann **SAS und Verbindungszeichenfolge generieren** aus. Das SAS-Token wird im Feld **SAS-Token** angezeigt, und kopieren Sie es dann.
1. Um eine Verbindung mit Ihrem Azure-Konto herzustellen, verwenden Sie PowerShell und führen das Cmdlet `Connect-AzAccount` aus.

    ```powershell
    Login-AzAccount
    Select-AzSubscription -SubscriptionId <subscription ID>
    ```

1. Verwenden Sie das Cmdlet `Start-AzSqlInstanceDatabaseLogReplay`, um den Protokollwiederherstellungsdienst für die Datenbank, die Sie wiederherstellen möchten, zu starten. Sie müssen den Namen der Ressourcengruppe, den Instanznamen, den Datenbanknamen, den Speichercontainer-URI und das SAS-Token angeben, das Sie zuvor kopiert haben.

```PowerShell
Import-Module Az.Sql

Start-AzSqlInstanceDatabaseLogReplay -ResourceGroupName "YourResourceGroupName" -InstanceName "YourInstanceName" -Name "YourDatabaseName" -StorageContainerUri "https://yourstorageaccount.blob.core.windows.net/yourcontainer" -StorageContainerSasToken "YourSasToken"
```

## Überwachen des Migrationsfortschritts

Sie können mit dem Cmdlet `Get-AzSqlInstanceDatabaseLogReplay` den Status des Protokollwiedergabediensts überwachen. Dieses Cmdlet gibt Informationen zum aktuellen Status des Dienstes zurück, einschließlich der zuletzt wiederhergestellten Protokollsicherungsdatei.

1. Führen Sie den folgenden PowerShell-Code aus.

```powershell
# Import the Az.Sql module
Import-Module Az.Sql

# Set the resource group name, instance name, and database name
$resourceGroupName = "YourResourceGroupName"
$instanceName = "YourInstanceName"
$databaseName = "YourDatabaseName"

# Get the log replay status
$logReplayStatus = Get-AzSqlInstanceDatabaseLogReplay -ResourceGroupName $resourceGroupName -InstanceName $instanceName -Name $databaseName

# Display the log replay status
$logReplayStatus | Format-List
```

## Durchführen der Migrationsübernahme

Nachdem die vollständige Datenbanksicherung in der verwalteten Azure SQL-Datenbank-Zielinstanz wiederhergestellt wurde, steht die Datenbank für eine Migrationsübernahme zur Verfügung.

1. Wenn Sie die Onlinedatenmigration abschließen möchten, klicken Sie auf **Übernahme starten**.
1. Beenden Sie den gesamten eingehenden Datenverkehr für Quelldatenbanken.
1. Verwenden Sie die Sicherung des Protokollfragments, machen Sie die Sicherungsdatei auf der SMB-Netzwerkfreigabe verfügbar, und warten Sie, bis diese letzte Transaktionsprotokollsicherung wiederhergestellt wurde.
1. Für **Ausstehende Änderungen** wird nun „0“ angezeigt.
1. Aktivieren Sie das Kontrollkästchen **Bestätigen**, und klicken Sie anschließend auf **Anwenden**.

    ![Bildschirm für Migrationsübernahme](../media/3-migration-cutover-screen.png)

1. Wenn als Status der Datenmigration **Abgeschlossen** angezeigt wird, stellen Sie eine Verbindung zwischen Ihren Anwendungen und der neuen verwalteten Azure SQL-Datenbank-Zielinstanz her.