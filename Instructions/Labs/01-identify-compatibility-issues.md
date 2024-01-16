---
lab:
  title: Identifizieren von Kompatibilitätsproblemen bei der SQL-Migration
---

# Identifizieren von Kompatibilitätsproblemen bei der SQL-Migration

In unserem Szenario wurden Sie aufgefordert, die Bereitschaft einer SQL Server-Legacydatenbank für die Migration zu Azure SQL-Datenbank zu bewerten. Ihre Aufgabe besteht darin, eine Bewertung der Legacydatenbank durchzuführen und potenzielle Kompatibilitätsprobleme oder Änderungen zu identifizieren, die vor der Migration vorgenommen werden müssen. Sie sollten auch das Schema der Datenbank überprüfen und alle Features oder Konfigurationen identifizieren, die in Azure SQL-Datenbank nicht unterstützt werden.

Diese Übung dauert ungefähr **15** Minuten.

> **Hinweis**: Um diese Übung abzuschließen, benötigen Sie Zugriff auf ein Azure-Abonnement, um Azure-Ressourcen zu erstellen. Wenn Sie kein Azure-Abonnement besitzen, können Sie ein [kostenloses Konto](https://azure.microsoft.com/free/?azure-portal=true) erstellen, bevor Sie beginnen.

## Vorbereitung

Um diese Übung durchführen zu können, müssen Sie sicherstellen, dass Sie die folgenden Voraussetzungen erfüllen, bevor Sie fortfahren:

- Sie benötigen SQL Server 2019 oder eine neuere Version zusammen mit der [**AdventureWorksLT**](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)-Datenbank, die mit Ihrer spezifischen SQL Server-Instanz kompatibel ist.
- Laden Sie [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio) herunter und installieren Sie es. Wenn sie bereits installiert ist, aktualisieren Sie es, um sicherzustellen, dass Sie die neueste Version verwenden.
- Ein*e SQL-Benutzer*in mit Lesezugriff auf die Quelldatenbank.

## Wiederherstellen einer SQL Server-Datenbank und Ausführen eines Befehls

1. Wählen Sie die Windows-Startschaltfläche aus, und geben Sie „SSMS“ ein. Wählen Sie **Microsoft SQL Server Management Studio 18** aus der Liste aus.  

1. Wenn SSMS geöffnet wird, beachten Sie, dass das Dialogfeld **Mit Server verbinden** mit dem Standardinstanznamen vorausgefüllt ist. Wählen Sie **Verbinden**.

1. Wählen Sie den Ordner **Datenbanken**, und dann **Neue Abfrage**.

1. Kopieren Sie im neuen Abfragefenster die folgende T-SQL und fügen Sie sie ein. Führen Sie die Abfrage aus, um die Datenbank wiederherzustellen.

    ```sql
    RESTORE DATABASE AdventureWorksLT
    FROM DISK = 'C:\LabFiles\AdventureWorksLT2019.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorksLT2019_Data' 
            TO 'C:\LabFiles\AdventureWorksLT2019.mdf',
          MOVE 'AdventureWorksLT2019_Log'
            TO 'C:\LabFiles\AdventureWorksLT2019.ldf';
    ```

    > **Hinweis**: Stellen Sie sicher, dass der Name der Datenbanksicherungsdatei und der Pfad im obigen Beispiel Ihrer tatsächlichen Sicherungsdatei entsprechen. Ist dies nicht der Fall, kann der Befehl fehlschlagen.

1. Nach Abschluss der Wiederherstellung sollte eine erfolgreiche Meldung angezeigt werden.

1. Führen Sie den folgenden Befehl für die **AdventureWorksLT**-Datenbank in der SQL Server-Instanz aus.

```sql
ALTER TABLE [SalesLT].[Customer] ADD [Next] VARCHAR(5);
```

## Installieren und Ausführen der Azure-Migrationserweiterung für Azure Data Studio

Folgen Sie diesen Schritten, um die Migrationserweiterung zu installieren. Wenn die Azure-Migrationserweiterung bereits installiert ist, können Sie diese Schritte überspringen.

1. Öffnen Sie den Erweiterungs-Manager in Azure Data Studio. s

1. Suchen Sie nach ***Azure SQL Migration***, und installieren Sie die Erweiterung. Nach der Installation befindet sich die Azure SQL Migrationserweiterung in der Liste der installierten Erweiterungen.

1. Wählen Sie das Symbol **Verbindungen**, und wählen Sie dann **Neue Verbindung**. 

1. Geben Sie auf der neuen Registerkarte **Verbindung** den Servernamen ein. Wählen Sie **Optional (False)** für die Option **Encrypt**.

1. Wählen Sie **Verbinden**. 

1. Um die Azure-Migrationserweiterung zu starten, klicken Sie einfach mit der rechten Maustaste auf den Namen der Quellinstanz und wählen **Verwalten**. 

1. Wählen Sie im Servermenü unter **Allgemein** die Option **Azure SQL Migration** aus. Dadurch gelangen Sie zur Standardseite der Azure SQL-Migrationserweiterung.

    > **Hinweis**: Wenn Sie die Option **Azure SQL Migration** im Servermenü nicht sehen können oder die Azure SQL Migration-Seite nicht geladen wird, öffnen Sie Azure Data Studio erneut.

## Ausführen der Kompatibilitätsbewertung

Die Kompatibilitätsbewertung hilft bei der Identifizierung potenzieller Migrationsprobleme und bietet detaillierte Anleitungen zum Beheben dieser Probleme, bevor der Migrationsprozess beginnt. Dies kann viel Zeit und Ressourcen sparen. 

Sie führen die Azure-Migrationserweiterung für Azure Data Studio aus, führen die Kompatibilitätsprüfung durch und zeigen dann die Ergebnisse für ein Azure SQL Datenbank-Ziel an.

1. Wählen Sie im Azure SQL Migration-Dashboard die Option **Zu Azure SQL migrieren** aus, um den Migrations-Assistenten zu öffnen.

1. Wählen Sie unter **Schritt 1: Datenbanken für die Bewertung** die *AdventureWorks*-Datenbank und dann **Weiter** aus.

1. Warten Sie unter **Schritt 2: Bewertungsergebnisse und Empfehlungen**, bis die Bewertung abgeschlossen wurde.

## Überprüfen der Bewertungsergebnisse

Sie können nun die von der Migrationserweiterung generierten Empfehlungen überprüfen.

1. Unter **Schritt 2: Bewertungsergebnisse und Empfehlungen** wählen Sie **Azure SQL-Datenbank** als Zielplattform aus.

1. Wählen Sie unten auf der Seite **Ansicht/Auswählen** aus, um die Bewertungsergebnisse anzuzeigen. 

1. Wählen Sie die *AdventureWorks*-Datenbank aus. Nehmen Sie sich einen Moment Zeit, um die Bewertungsergebnisse auf der rechten Seite zu überprüfen.
    
    > **Hinweis:** Wir sehen, dass die zuvor hinzugefügte Spalte `Next` markiert wurde, da sie zu einem Fehler in der Azure SQL-Datenbank führen kann.

1. Wählen Sie **Abbrechen** aus, und wählen Sie stattdessen **Azure SQL Managed Instance** als **Azure SQL**-Zielplattform aus.
    
    > **Hinweis:** Die Spalte `Next` ist für Azure SQL Managed Instance nicht mehr gekennzeichnet, woran liegt das? 
    >
    >Das bedeutet, dass die Spalte `Next` sicher auf Azure SQL Managed Instance verwendet werden kann.

1. Wählen Sie **Bewertungsbericht speichern**, um den Bericht in einem JSON-Format zu speichern.

1. Nehmen Sie sich einen Moment Zeit, um die JSON-Datei und ihre Eigenschaften zu überprüfen.

## Beheben des Problems

1. Führen Sie den folgenden T-SQL-Befehl für die *AdventureWorks*-Datenbank aus.

    ```sql
    ALTER TABLE [SalesLT].[Customer] DROP COLUMN [Next];
    ```

1. Gehen Sie zurück zur Seite **Schritt 2: Bewertungsergebnisse und Empfehlungen** im Assistenten und wählen Sie **Bewertung aktualisieren**.

1. Wählen Sie **Azure SQL-Datenbank** als **Azure SQL**-Zielplattform.

1. Wählen Sie **Anzeigen/Auswählen** aus, um die Bewertungsergebnisse anzuzeigen.

    > **Hinweis:** Das Problem ist nicht mehr gekennzeichnet.

Sie haben gelernt, wie Sie die Bereitschaft einer SQL Server-Datenbank für die Migration zu Azure SQL-Datenbank bewerten können. Indem Sie Kompatibilitätsprobleme beheben und wichtige Schemaänderungen vornehmen oder diese melden, haben Sie einen wichtigen Schritt unternommen, um potenzielle technische Probleme zu beheben, die in Zukunft in Azure SQL-Datenbank auftreten könnten.
