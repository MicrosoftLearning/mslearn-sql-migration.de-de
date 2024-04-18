---
lab:
  title: Migrieren von SQL Server-Datenbanken zu einer Azure SQL-Datenbank
---

# Migrieren von SQL Server-Datenbanken zu einer Azure SQL-Datenbank

In dieser Übung erfahren Sie, wie Sie mithilfe der Azure-Migrationserweiterung für Azure Data Studio eine SQL Server-Datenbank zu Azure SQL-Datenbank migrieren. Sie beginnen, indem Sie die Azure-Migrationserweiterung für Azure Data Studio installieren und starten. Anschließend führen Sie eine Offlinemigration einer SQL Server-Datenbank zu einer Azure SQL-Datenbank durch. Außerdem erfahren Sie, wie Sie den Migrationsprozess auf dem Azure-Portal überwachen.

Diese Übung dauert ca. **45** Minuten.

> **Hinweis**: Um diese Übung abzuschließen, benötigen Sie Zugriff auf ein Azure-Abonnement, um Azure-Ressourcen zu erstellen. Wenn Sie kein Azure-Abonnement besitzen, können Sie ein [kostenloses Konto](https://azure.microsoft.com/free/?azure-portal=true) erstellen, bevor Sie beginnen.

## Vorbereitung

Um diese Übung auszuführen, benötigen Sie Folgendes:

| Element | Beschreibung |
| --- | --- |
| **Zielserver** | Ein Azure SQL-Datenbank-Server. Wir erstellen ihn während dieser Übung.|
| **Zieldatenbank** | Eine Datenbank auf dem Azure SQL-Datenbank-Server. Wir erstellen ihn während dieser Übung.|
| **Quellserver** | Eine Instanz von SQL Server 2019 oder eine [neuere Version](https://www.microsoft.com/en-us/sql-server/sql-server-downloads), die auf einem Server Ihrer Einstellung installiert ist. |
| **Quelldatenbank** | Die einfache [AdventureWorks](https://learn.microsoft.com/sql/samples/adventureworks-install-configure)-Datenbank, die auf der SQL Server-Instanz wiederhergestellt werden soll. |
| **Azure Data Studio** | Installieren Sie [Azure Data Studio](https://learn.microsoft.com/sql/azure-data-studio/download-azure-data-studio) auf demselben Server, auf dem sich die Quelldatenbank befindet. Wenn sie bereits installiert ist, aktualisieren Sie es, um sicherzustellen, dass Sie die neueste Version verwenden. |
| **Microsoft.DataMigration**-Ressourcenanbieter | Stellen Sie sicher, dass das Abonnement für die Verwendung des Namespace **Microsoft.DataMigration** registriert ist. Wie Sie die Registrierung eines Ressourcenanbieters durchführen, erfahren Sie unter [Registrieren des Ressourcenanbieters](https://learn.microsoft.com/azure/dms/quickstart-create-data-migration-service-portal#register-the-resource-provider). |
| **Microsoft Integration Runtime** | Installieren Sie [Microsoft Integration Runtime](https://aka.ms/sql-migration-shir-download). |

## Wiederherstellen einer SQL Server-Datenbank

Stellen wir die Datenbank *AdventureWorksLT* auf der SQL Server-Instanz wieder her. Diese Datenbank dient als Quelldatenbank für diese Labübung. Sie können diese Schritte überspringen, wenn die Datenbank bereits wiederhergestellt wurde.

1. Wählen Sie die Windows-Startschaltfläche aus, und geben Sie „SSMS“ ein. Wählen Sie **Microsoft SQL Server Management Studio 18** aus der Liste aus.  

1. Wenn SSMS geöffnet wird, beachten Sie, dass das Dialogfeld **Mit Server verbinden** mit dem Standardinstanznamen vorausgefüllt ist. Wählen Sie **Verbinden** aus.

1. Wählen Sie den Ordner **Datenbanken**, und dann **Neue Abfrage**.

1. Kopieren Sie im neuen Abfragefenster die folgende T-SQL und fügen Sie sie ein. Vergewissern Sie sich, dass der Name und der Pfad der Datenbanksicherungsdatei mit Ihrer tatsächlichen Sicherungsdatei übereinstimmen. Ist dies nicht der Fall, schlägt der Befehl fehl. Führen Sie die Abfrage aus, um die Datenbank wiederherzustellen.

    ```sql
    RESTORE DATABASE AdventureWorksLT
    FROM DISK = 'C:\<FolderName>\AdventureWorksLT2019.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorksLT2019_Data' 
            TO 'C:\<FolderName>\AdventureWorksLT2019.mdf',
          MOVE 'AdventureWorksLT2019_Log'
            TO 'C:\<FolderName>\AdventureWorksLT2019.ldf';
    ```

    > **Hinweis:** Stellen Sie sicher, dass Sie über die einfache [AdventureWorks-Sicherungsdatei](https://learn.microsoft.com/sql/samples/adventureworks-install-configure#download-backup-files) auf dem SQL Server-Computer verfügen, bevor Sie den T-SQL-Befehl ausführen.

1. Nach Abschluss der Wiederherstellung sollte eine erfolgreiche Meldung angezeigt werden.

## Registrieren des Microsoft.DataMigration-Namespace

Überspringen Sie diese Schritte, wenn der **Microsoft.DataMigration**-Namespace bereits in Ihrem Abonnement registriert ist.

1. Suchen Sie im Azure-Portal im oberen Suchfeld nach **Abonnement** und wählen Sie dann **Abonnements**. Wählen Sie Ihr Abonnement auf der Seite **Abonnements** aus.

1. Wählen Sie auf Ihrer Abonnement-Seite **Ressourcenanbieter** unter **Einstellungen** aus. Suchen Sie im oberen Suchfeld nach **Microsoft.DataMigration** und wählen Sie dann **Microsoft.DataMigration** aus. 

    > **Hinweis**: Wenn die Seitenleiste **Ressourcenanbieterdetails** geöffnet wird, können Sie sie schließen.

1. Wählen Sie **Registrieren** aus.

## Bereitstellen einer Azure SQL-Datenbank

Lassen Sie uns eine Azure SQL-Datenbank einrichten, die als Zielumgebung dienen wird.

1. Suchen Sie im Azure-Portal im oberen Suchfeld nach **SQL-Datenbanken** und wählen Sie dann **SQL-Datenbanken**.

1. Wählen Sie im Bereich **SQL-Datenbanken** die Option **+ Erstellen**.

1. Wählen Sie auf der Seite **SQL-Datenbank erstellen** die folgenden Optionen.

    - **Abonnement:** &lt;Ihr Abonnement&gt;
    - **Ressourcengruppe:**&lt;Ihre Ressourcengruppe&gt;
    - **Datenbankname**: AdventureWorksLT
    - **Server:** Wählen Sie den Link **Neuen erstellen**. Geben Sie die Serverdetails auf der Seite **SQL-Datenbankserver erstellen** an.
        - **Servername:**&lt;Wählen Sie einen Servernamen&gt; aus. Der Servername muss global eindeutig sein.
        - **Standort:**&lt;Dieselbe Region wie Ihre Ressourcengruppe&gt;
        - **Authentifizierungsmethode**: Verwenden Sie SQL-Authentifizierung
        - **Serveradministratoranmeldung:** sqladmin
        - **Kennwort**: &lt;Ihr Kennwort&gt;
        - **Kennwort bestätigen:**&lt;Ihr Kennwort&gt;
        
            **Hinweis:** Notieren Sie sich diesen Servernamen und die Anmeldeinformationen. Sie werden sie in späteren Aufgaben verwenden.

    - **Möchten Sie einen Pool für elastische SQL-Datenbanken verwenden?** Nein
    - **Workloadumgebung:** Produktion

1. Wählen Sie auf **Compute + Speicher** die Option **Datenbank konfigurieren** aus. Auf der Seite **Konfigurieren** wählen Sie im Dropdown-Menü **Dienstebene** die Option **Basic** und dann **Anwenden**.

1. Für die Option **Sicherungsspeicherredundanz** behalten Sie den Standardwert bei: **Georedundanter Sicherungsspeicher**. Klicken Sie auf **Überprüfen + erstellen**.

1. Überprüfen Sie die Einstellungen und wählen Sie dann **Erstellen**.

1. Klicken Sie nach Abschluss der Bereitstellung auf **Zu Ressource wechseln**.

## Aktivieren des Zugriffs auf eine Azure SQL-Datenbank

Lassen Sie uns den Zugriff auf eine Azure SQL-Datenbank ermöglichen, damit wir über die Clienttools eine Verbindung damit herstellen können.

1. Wählen Sie auf der Seite **SQL-Datenbank** den Abschnitt **Übersicht** und wählen Sie dann den Link für den Servernamen im oberen Abschnitt:

1. Wählen Sie auf dem Blatt **SQL Server** unter dem Abschnitt **Sicherheit** die Option **Netzwerke**.

1. Wählen Sie auf der Registerkarte **Öffentlicher Zugriff** **Ausgewählte Netzwerke** aus. 

1. Wählen Sie unter **Firewallregeln** die Option **+ Client-IPv4-Adresse hinzufügen**.

1. Überprüfen Sie unter **Ausnahmen** die Eigenschaft **Zugriff von Azure-Diensten und -Ressourcen auf diesen Server erlauben**. 

1. Wählen Sie **Speichern**.

## Herstellen einer Verbindung mit einer Azure SQL-Datenbank in Azure Data Studio

Bevor Sie mit der Azure-Migrationserweiterung beginnen, stellen wir eine Verbindung mit der Zieldatenbank her.

1. Starten Sie Azure Data Studio.

1. Wählen Sie **Verbindungen**, dann **Verbindung hinzufügen**.

1. Füllen Sie die Felder **Verbindungsdetails** mit dem Namen des SQL-Servers und anderen Informationen aus.

    > **Hinweis**: Geben Sie den Namen des zuvor erstellten SQL Servers ein. Das Format sollte im Format von **<server>.database.windows.net** sein.

1. Wählen Sie unter **Authentifizierungstyp** die Option **SQL-Anmeldung** und geben Sie den Benutzernamen und das Kennwort ein.

1. Wählen Sie **Verbinden** aus.

## Installieren und Ausführen der Azure-Migrationserweiterung für Azure Data Studio

Folgen Sie diesen Schritten, um die Migrationserweiterung zu installieren. Wenn die Erweiterung bereits installiert ist, können Sie diese Schritte überspringen.

1. Öffnen Sie den Erweiterungs-Manager in Azure Data Studio.

1. Suchen Sie nach ***Azure SQL Migration***, und wählen Sie die Erweiterung aus.

1. Installieren Sie die Erweiterung. Nach der Installation befindet sich die Azure SQL Migrationserweiterung in der Liste der installierten Erweiterungen.

1. Stellen Sie in Azure Data Studio eine Verbindung mit einer SQL Server-Instanz her. Wählen Sie auf der Registerkarte „Neue Verbindung“ die Option **Optional (False)** für die Option **Verschlüsseln**.

1. Um die Azure-Migrationserweiterung zu starten, klicken Sie mit der rechten Maustaste auf den Namen der SQL Server-Instanz und wählen Sie **Verwalten**, um auf das Dashboard und die Landing Page der Azure SQL Migrationserweiterung zuzugreifen.

    > **Hinweis**: Wenn **Azure SQL Migration** in der Seitenleiste des Server-Dashboards nicht sichtbar ist, öffnen Sie Azure Data Studio erneut.
 
## Durchführen einer Offlinemigration einer SQL Server-Datenbank zu einer Azure SQL-Datenbank

Wir sind jetzt bereit, die Daten zu migrieren. Folgen Sie diesen Schritten, um eine Offlinemigration mit Azure Data Studio durchzuführen.

1. Starten Sie den Assistenten **Zu Azure SQL migrieren** innerhalb der Erweiterung in Azure Data Studio, und wählen Sie dann **Zu Azure SQL migrieren**.

1. Wählen Sie in **Schritt 1: Datenbanken für die Bewertung** die Datenbank *AdventureWorksLT* und dann **Weiter** aus.

1. In **Schritt 2: Bewertungszusammenfassung und SKU-Empfehlungen** warten Sie, bis die Bewertung abgeschlossen ist, und sehen Sie sich dann die Ergebnisse an. Wählen Sie **Weiter** aus.

1. In **Schritt 3: Zielplattform und Bewertungsergebnisse** wählen Sie **Azure SQL-Datenbank** als Zieltyp aus. Wählen Sie nach der Überprüfung der Bewertungsergebnisse **Weiter** aus.

1. In **Schritt 4: Azure SQL-Ziel**, wenn das Konto noch nicht verknüpft ist, fügen Sie ein Konto hinzu. Wählen Sie dazu einfach den Link **Konto verknüpfen** aus. Wählen Sie dann ein Azure-Konto, einen Microsoft Entra-Mandanten, ein Abonnement, einen Standort, eine Ressourcengruppe, einen Azure SQL-Datenbankserver und die Anmeldedaten der Azure SQL-Datenbank aus.

1. Wählen Sie **Verbinden** aus, und wählen Sie dann die Datenbank *AdventureWorksLT* als **Zieldatenbank** aus. Wählen Sie **Weiter** aus.

1. Wählen Sie in **Schritt 5: Azure Database Migration Service** den Link **Neu erstellen** aus, um mithilfe des Assistenten einen neuen Azure Database Migration Service zu erstellen. Führen Sie die vom Assistenten bereitgestellten Schritte aus, um eine neue selbst gehostete Integration Runtime einzurichten. Wenn Sie bereits einen solchen erstellt haben, können Sie ihn wiederverwenden.

1. Geben Sie in **Schritt 6: Konfiguration der Datenquelle** die Anmeldeinformationen ein, um über die selbstgehostete Integration Runtime eine Verbindung mit der SQL Server-Instanz herzustellen.

1. Wählen Sie **Bearbeiten** für die *AdventureWorksLT*-Datenbank aus. 

1. Aktivieren Sie die Option **Schema auf Ziel migrieren**, und vergewissern Sie sich, dass alle Tabellen auf der Registerkarte **Im Ziel nicht vorhanden** ausgewählt sind. Wählen Sie **Aktualisieren**.

1. Wählen Sie **Überprüfung ausführen**.

    ![Screenshot des Schritts „Überprüfung ausführen“ in der Azure-Migrationserweiterung für Azure Data Studio.](../media/3-run-validation.png) 

1. Wählen Sie nach Abschluss der Überprüfung die Option **Fertig** aus.

1. Wählen Sie **Weiter** aus.

1. Wählen Sie in **Schritt 7: Zusammenfassung** die Option **Migration starten** aus.

1. Wählen Sie **Datenbankmigrationen in Bearbeitung** im Migrationsdashboard, um den Migrationsstatus anzuzeigen. 

    ![Screenshot: Migrationsdashboard in der Azure-Migrationserweiterung für Azure Data Studio.](../media/3-data-migration-dashboard.png)

1. Wählen Sie den Namen der *AdventureWorks*-Datenbank aus, um weitere Details anzuzeigen.

    ![Screenshot: Migrationsdetails in der Azure-Migrationserweiterung für Azure Data Studio.](../media/3-dashboard-sqldb.png)

1. Nachdem der Status **Erfolgreich** ist, navigieren Sie zum Zielserver und überprüfen Sie die Zieldatenbank. Überprüfen Sie das Datenbankschema und die migrierten Daten.

Sie haben erfahren, wie Sie die Migrationserweiterung installieren und das Datenbankschema mithilfe des Datenmigrations-Assistent generieren. Sie haben auch gelernt, wie Sie eine SQL Server-Datenbank mit der Azure-Migrationserweiterung für Azure Data Studio in eine Azure-SQL-Datenbank migrieren können. Nach Abschluss der Migration können Sie mit der Verwendung Ihrer neuen Azure SQL-Datenbank-Ressource beginnen. 

## Bereinigen

Wenn Sie in Ihrem eigenen Abonnement arbeiten, sollten Sie sich am Ende eines Projekts überlegen, ob Sie die erstellten Ressourcen noch benötigen. 

Wenn Ressourcen unnötig ausgeführt werden, kann dies zu zusätzlichen Kosten führen. Sie können Ressourcen einzeln oder die gesamte Gruppe von Ressourcen im [Azure-Portal](https://portal.azure.com?azure-portal=true) löschen.

## Weitere Informationen

Weitere Informationen über Azure SQL-Datenbank finden Sie unter [Was ist Azure SQL-Datenbank?](https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview)
