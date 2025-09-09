---
lab:
  title: Vergleichen der Azure-Kosten für die Migration
---

# Vergleichen der Azure-Kosten für die Migration

Der Azure-Preisrechner ist ein nützliches Tool, das Sie während eines Projekts für die Datenplattformmodernisierung verwenden können, um die Kosten verschiedener Azure-Dienste und Migrationsansätze zu schätzen.

In Ihrem globalen Einzelhandelsunternehmen sollen durch das Projekt für die Modernisierung der Datenplattform erhebliche Einsparungen erzielt werden, aber der Vorstand hat Sie gebeten, die Kosten verschiedener Azure-Migrationsoptionen Einsparungen so genau wie möglich zu schätzen.

Hier berechnen Sie die geschätzten Kosten für die Migration zu Azure mithilfe des [Azure-Preisrechners](https://azure.microsoft.com/en-us/pricing/calculator/).

Diese Übung dauert ungefähr **30** Minuten.

## Berechnen der geschätzten Azure-Kosten

1. Öffnen Sie eine neue Browserregisterkarte, und navigieren Sie zu [Azure-Preisrechner](https://azure.microsoft.com/en-us/pricing/calculator/).
1. Mit dem Azure-Preisrechner können Sie die Kosten für Azure-Dienste schätzen. Wir berechnen die Kosten für die Migration Ihrer Datenbankworkload zu Azure.

## Hinzufügen des Datenbankworkloads

1. Suchen Sie im Abschnitt **Produkte** nach **Azure SQL-Datenbank**, und wählen Sie diese Option aus. (Sie können die Suche nutzen oder die Kategorie **Datenbanken** durchsuchen.)
1. Geben Sie im Konfigurationsbereich **Azure SQL-Datenbank**, der weiter unten auf der Seite angezeigt wird, die folgenden Werte ein:

    | Eigenschaft | Wert |
    | --- | --- |
    | Region | **USA, Osten** (bzw. Ihre bevorzugte Region) |
    | Typ | **Einzeldatenbank** |
    | Kaufmodell | **Virtueller Kern** |
    | Dienstebene | **Allgemeiner Zweck** |
    | Computeebene | **Bereitgestellt** |
    | Hardwaretyp | **Standard-Serie (Gen5)** |
    | Instanz | **4 virtuelle Kerne** |
    | Notfallwiederherstellung | **Primäres Replikat oder Geo-Replikat** |
    | Compute | **Lokal redundant** |
    | Storage | **32 GB** |
    | Sicherungsspeicher | **RA-GRS** |

1. Überprüfen Sie die monatliche Kostenschätzung, die für SQL-Datenbank angezeigt wird.

## Hinzufügen eines virtuellen Computers zum Vergleich

1. Kehren Sie zurück zum Abschnitt **Produkte**, suchen Sie nach **Virtuelle Computer**, und wählen Sie diese Option aus.
1. Geben Sie im Konfigurationsbereich **Virtuelle Computer** die folgenden Werte ein:

    | Eigenschaft | Wert |
    | --- | --- |
    | Region | **USA, Osten** (genau wie für die Datenbank) |
    | Betriebssystem | **Windows** |
    | Typ | **SQL Server** |
    | Tarif | **Standard** |
    | Instanz | **D4s v3** (4 vCPUs, 16 GB RAM) |
    | Virtuelle Computer | **1** |
    | Lizenz | **SQL Standard** |

1. Erweitern Sie **Verwaltete Datenträger**, und fügen Sie Folgendes hinzu:
   - Tarif: **SSD Premium**, **128 GB**

## Hinzufügen von Speicher für Sicherungen

1. Suchen Sie im Abschnitt **Produkte** nach **Speicherkonten**, und wählen Sie diese Option aus.
1. Geben Sie im Konfigurationsbereich **Speicherkonten** die folgenden Werte ein:

    | Eigenschaft | Wert |
    | --- | --- |
    | Region | **USA, Osten** |
    | Typ | **Blockblobspeicher** |
    | Leistung | **Standard** |
    | Speicherkontotyp | **Universell V2** |
    | Dateistruktur | **Flacher Namespace** |
    | Zugriffsebene | **Heiße Ebene** |
    | Redundanz | **LRS** |
    | Capacity | **1 TB** |

## Überprüfen und Vergleichen von Kosten

1. Überprüfen Sie die geschätzten monatlichen Kosten für alle Dienste, die Sie hinzugefügt haben:
   - **SQL-Datenbank:** Kosten für verwaltete Datenbankdienste
   - **Virtuelle Computer:** Infrastrukturkosten für SQL Server auf einer VM
   - **Speicherkonten**: Kosten für Sicherung und zusätzliche Speicher

1. Berücksichtigen Sie die folgenden Fragen, wenn Sie die Schätzungen überprüfen:
   - Wie hoch sind die Kosten im Vergleich zwischen SQL-Datenbank (PaaS) und SQL Server auf einer VM (IaaS)?
   - Was sind die Vor- und Nachteile von verwalteten Diensten gegenüber Infrastrukturdiensten?
   - Wie können diese Kosten mit Ihren Workloadanforderungen skaliert werden?

1. Sie können diese Schätzung speichern, indem Sie **Speichern** oder **Exportieren**, auswählen, um sie für Projektbeteiligte freizugeben.

## Erkunden von Kostenoptimierungsoptionen

1. Erkunden Sie in jeder Dienstkonfiguration verschiedene Optionen, um zu sehen, wie sich die Kosten auswirken:
   - **SQL-Datenbank:** Probieren Sie verschiedene Dienstebenen (Basic, Standard, Premium) und Computegrößen aus.
   - **Virtuelle Computer:** Vergleichen Sie verschiedene VM-Größen und Speicheroptionen.
   - **Speicher:** Vergleichen Sie verschiedene Redundanzoptionen und Zugriffsebenen.

1. Beachten Sie, wie sich das Ändern dieser Optionen auf Ihre monatliche Gesamtschätzung auswirkt.

Sie haben den Azure-Preisrechner verwendet, um die Kosten für die Migration des Buchhaltungsservers der Adatum Corporation und die zugehörigen Datenbanken zu verschiedenen Azure-Diensten zu schätzen. Dadurch erhalten Sie eine Grundlage für den Vergleich von IaaS- und PaaS-Optionen (Infrastructure-as-a-Service bzw. Platform-as-a-Service) und die Planung Ihres Migrationsbudgets.

