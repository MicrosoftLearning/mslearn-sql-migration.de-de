---
lab:
  title: Vergleichen der lokalen Azure-Kosten für die Migration
---

# Vergleichen der lokalen Azure-Kosten für die Migration

Die Gesamtkosten (Total Cost of Ownership, TCO) sind ein Tool, das Sie während eines Projekts zur Modernisierung der Datenplattform verwenden können, um den Kostenunterschied zu ermitteln, der mit der Migration erzielt werden kann.

In Ihrem globalen Einzelhandelsunternehmen sollen durch die Modernisierung der Datenplattform erhebliche Einsparungen erzielt werden, aber der Vorstand hat Sie gebeten, die Einsparungen so genau wie möglich zu schätzen.

Hier berechnen Sie die Gesamtkosten der Migration zu Azure mithilfe des Gesamtkostenrechners.

Diese Übung dauert ungefähr **30** Minuten.

## Berechnen der Gesamtbetriebskosten (TCO)

1. Öffnen Sie eine neue Browserregisterkarte, und navigieren Sie zu [Azure-Gesamtkostenrechner](https://azure.microsoft.com/pricing/tco/calculator/).
1. Löschen Sie unter der Option zum **Definieren Ihrer Workloads** alle vorhandenen Workloads im Abschnitt **Server**.

## Hinzufügen des Datenbankworkloads

1. Wählen Sie unter **Datenbanken** die Option **+ Datenbank hinzufügen** aus.
1. Geben Sie in das Textfeld **Name** den Text **Ressourcenerfassung** ein.
1. Wählen Sie im Abschnitt **Quelle** die folgenden Werte aus:

    | Eigenschaft | Wert |
    | --- | --- |
    | Datenbank | **Microsoft SQL Server** |
    | Lizenz | **Enterprise** |
    | Environment | **Physische Server** |
    | Betriebssystem | **Windows** |
    | Lizenz des Betriebssystems | **Datacenter** |
    | Server | **1** |
    | Prozessoren pro Server | **1** |
    | Kern(e) pro Prozessor | **4** |
    | RAM (GB) | **64** |
    | Optimieren für | **CPU** |
    | SQL Server 2008/2008R2 | **Umschalten, um diesen Wert auszuwählen** |

1. Wählen Sie im Abschnitt **Ziel** die folgenden Werte aus:

    | Eigenschaft | Wert |
    | --- | --- |
    | Dienst | **SQL Server-VM** |
    | Datenträgertyp | **SSD** |
    | IOPS | **5000** |
    | SQL Server-Speicher | **32 GB** |
    | SQL Server-Sicherung | **32 GB** |

    > [!NOTE]
    > SSD-Datenträger (Solid State Drive) werden für Produktionsworkloads in Azure empfohlen.

## Hinzufügen von Speicher- und Netzwerkworkloads

1. Wählen Sie unter **Speicher** die Option **+ Speicher hinzufügen** aus.
1. Geben Sie in das Textfeld **Name** den Text **Ressourcenerfassung lokaler Datenträger** ein, und geben Sie dann die folgenden Werte ein:

    | Eigenschaft | Wert |
    | --- | --- |
    | Speichertyp | **Lokaler Datenträger/SAN** |
    | Datenträgertyp | **Festplattenlaufwerk** |
    | Capacity | **3 TB** |
    | Backup | **1 TB** |
    | Archivieren | **0 TB** |

1. Wählen Sie unter **Netzwerk** in den Steuerelementen **Ausgehende Bandbreite** die Option **1 GB** aus.
1. Wählen Sie unten auf der Seite die Option **Weiter** aus.

## Anpassen von Annahmen

1. Wählen Sie im Abschnitt **Anpassen von Annahmen** in der Liste **Währung** Ihre bevorzugte Währung aus.
1. Wählen Sie unter **Software Assurance-Abdeckung (bietet Azure-Hybridvorteil)** die Option für die **Windows Server Software Assurance-Abdeckung** aus, indem Sie den Umschalter aktivieren.
1. Wählen Sie **SQL Server Software Assurance-Abdeckung** aus, indem Sie den Umschalter aktivieren.

    > [!NOTE]
    > Sie können die Links im Abschnitt **Software Assurance** verwenden, um mehr über die verfügbare Software Assurance zu erfahren. 

1. Stellen Sie unter **Georedundanter Speicher (GRS)** sicher, dass **GRS repliziert Ihre Daten in eine sekundäre Region, die Hunderte von Kilometern von der primären Region entfernt ist** nicht aktiviert ist.
1. Stellen Sie unter **Kosten virtueller Computer** sicher, dass **Aktivieren Sie dies für den Rechner, um virtuelle Computern der Bs-Serie nicht zu empfehlen** aktiviert ist.

    > [!NOTE]
    > Die virtuellen Computer der B-Serie verfügen nicht über das Arbeitsspeicher-zu-vCore-Verhältnis (virtueller Kern) von 8, das für SQL Server-Workloads empfohlen wird.

1. Geben Sie unter **Stromkosten**, im Textfeld **Preis pro KW-Stunde** einen realistischen Wert für Ihren Standort ein.

    > [!NOTE]
    > Ungefähre Strompreise finden Sie unter [Globale Strompreise](https://www.statista.com/statistics/263492/electricity-prices-in-selected-countries/). Diese Preise sind in USD ($) angegeben. Rechnen Sie sie in einen ungefähren Wert in Ihrer bevorzugten Währung um.

1. Belassen Sie unter **Speicherkosten** alle Werte auf ihren Standardwerten oder passen Sie sie an, wenn sie unangemessen erscheinen.
1. Belassen Sie unter **IT-Arbeitskosten** alle Werte auf ihren Standardwerten oder passen Sie sie an, wenn sie unangemessen erscheinen.
1. Erweitern Sie unter **Sonstige Annahme** jeden Abschnitt und betrachten Sie die zugehörigen Kosten.
1. Wählen Sie unten auf der Seite die Option **Weiter** aus.

## Untersuchen des 5-Jahre-Berichts

1. Beachten Sie auf der Seite **Bericht anzeigen**, dass der **Zeitrahmen** standardmäßig auf **5 Jahre** festgelegt ist.
1. Scrollen Sie im Bericht nach unten, und untersuchen Sie die geschätzte Aufschlüsselung der Kosten für lokale Systeme und Azure. Notieren Sie sich diese Informationen.

    - Was ist die wichtigste Kostenkomponente der lokalen Umgebung?
    - Was sind die größten Kosteneinsparungen, wenn Sie sich für eine Migration zu Azure entscheiden?

1. Erweitern Sie jeden Abschnitt der Reihe nach, und untersuchen Sie die Aufschlüsselung der Kosten.

## Untersuchen des 3-Jahre-Berichts

1. Scrollen Sie zum oberen Rand der Seite, und wählen Sie dann im Textfeld **Zeitrahmen** die Option **3 Jahre** aus.
1. Scrollen Sie im Bericht nach unten, und untersuchen Sie die geschätzte Aufschlüsselung der Kosten für lokale Systeme und Azure. Notieren Sie sich diese Informationen.

    - Was ist die wichtigste Kostenkomponente der lokalen Umgebung?
    - Was sind die größten Kosteneinsparungen, wenn Sie sich für eine Migration zu Azure entscheiden?

1. Erweitern Sie jeden Abschnitt der Reihe nach, und untersuchen Sie die Aufschlüsselung der Kosten.

Sie haben den Azure-Gesamtkostenrechner verwendet, um Kostenunterschiede zwischen lokalen und Azure-Bereitstellungen für den Buchhaltungsserver der Adatum Corporation und die zugehörigen Datenbanken zu ermitteln.
