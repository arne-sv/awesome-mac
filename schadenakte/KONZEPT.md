# SchadenAkte – Konzept für eine Schadenbearbeitungs-Software für Sachverständige

`index.html` ist ein **klickbarer Prototyp**. Er läuft nur im Browser und speichert in localStorage. Er zeigt die Abläufe und die Assistenzlogik, ist aber **kein Produktivsystem**. Dieses Dokument beschreibt, was für ein echtes System nötig ist.

## 1. Was der Prototyp zeigt

| Bereich | Umsetzung im Prototyp |
|---|---|
| Auftragsanlage | Manuell, oder aus einer erkannten Auftrags-E-Mail (Schaden-Nr., VN, Adresse, Telefon und Sparte werden per Regex extrahiert) |
| Aktenübersicht | Stammdaten, Fortschritt, offene Aufgaben, Verlauf |
| Workflows | Vorlage je Sparte (Gebäude, Hausrat, Inhalt, Betriebsunterbrechung, Produkthaftpflicht, Transport, Marine/Binnenschiff). Pro Akte frei änderbar: abhaken, verschieben, löschen, ergänzen, Frist ändern |
| Spartenmasken | Gemeinsame Basisfelder plus spartenspezifische Felder (z. B. CMR-Daten, Schiffsdaten/ENI, BU-Haftzeit, Produkt/Inverkehrbringen). Pflichtfelder steuern die Assistenz |
| Beteiligte | Rollen je Akte (Anspruchsteller, Anwalt, Frachtführer, Werft, Steuerberater …). Ihre Mailadressen und Zeichen dienen der automatischen Mail-Zuordnung |
| Berechnungen | Unterversicherung (§ 75 VVG), Haftungshöchstbetrag CMR/HGB/CMNI in SZR und €, BU-Unterbrechungsschaden (vereinfachtes Umsatzverfahren), ProdHaftG-Selbstbeteiligung |
| Fristen | Automatisch aus den Maskendaten: Schadenanzeige und Verjährung im Frachtrecht, Verjährung und Erlöschen nach ProdHaftG und BGB. Übersicht über alle Akten, Erinnerung ab 30 Tagen vor Ablauf |
| Verknüpfte Akten | Ein Ereignis, mehrere Sparten (z. B. Brand → Inhalt + BU), Stammdaten und Beteiligte werden übernommen |
| E-Mail | Import echter `.eml`-Dateien per Drag & Drop (MIME, Base64/QP, Umlaute, Anhänge). Zuordnung über Aktenzeichen, Schaden-Nr., Schiffsname/ENI, Zeichen und Absender von Beteiligten. Neue Aufträge werden erkannt, die Sparte wird per Stichwortbewertung bestimmt, Schaden-Nr., VN, Adresse und Telefon werden extrahiert. Daraus entsteht die Aufgabe „Auftragsbestätigung“ mit fertigem Entwurf |
| Telefonate | Telefonnotiz mit optionaler Wiedervorlage, die als Workflow-Schritt eingefügt wird |
| Termine & Route | Termine je Akte. Tagesroute per Nächster-Nachbar-Verfahren, Warnung bei Konflikt mit den Uhrzeiten, Übergabe an Google Maps |
| Gutachten-Maske | Strukturierte Felder, die auswertbar sind, und PDF per Knopfdruck |
| Angebots-/Rechnungsprüfung | Positionen mit gefordertem und anerkanntem EP, Kürzung absolut und in %, Prüfbericht als PDF, Übernahme der Summe ins Gutachten |
| Assistenz | Regel-Engine erzeugt Aufgaben **ohne manuelle Anlage**: nächster Workflow-Schritt, fehlender Ortstermin, Terminvorbereitung, fehlendes Protokoll, offene Mails, offene Prüfungen, fehlende Pflichtfelder vor der Gutachtenfrist, Stillstand seit 7 Tagen, nicht zugeordnete Mails |

## 2. Was für den Produktivbetrieb fehlt (ehrliche Liste)

1. **Backend und Datenbank** (z. B. PostgreSQL) mit Mandantenfähigkeit, Rollen und Rechten sowie Audit-Log.
2. **Echte E-Mail-Anbindung**: Microsoft Graph (M365) oder IMAP/SMTP. Dazu gehören Anhänge, Threads, Versand aus der Akte und die Ablage als revisionssicheres `.eml`.
3. **Versicherer-Schnittstellen**: Viele Aufträge kommen über Portale oder Datenaustausch, z. B. GDV-Formate, Claims-Plattformen oder Kfz-Kalkulationssysteme. Das ist **projektentscheidend** und muss vor jeder Entwicklung je Auftraggeber geklärt werden.
4. **Kalender**: Abgleich mit Outlook/Google in beide Richtungen, Terminbestätigung an den VN per Mail oder SMS.
5. **Routenplanung**: echtes Geocoding und echte Fahrzeiten (z. B. OpenRouteService, HERE oder Google), Zeitfenster der Termine als Nebenbedingung.
6. **Telefonie**: CTI-Anbindung (z. B. Placetel, sipgate, Teams). Eingehende Rufnummer wird der Akte zugeordnet, Anrufnotiz wird automatisch angelegt.
7. **Dokumente**: Fotos vom Ortstermin (mobil, offlinefähig), Fotodokumentation im Gutachten, Vorlagen pro Auftraggeber, DOCX und PDF.
8. **Recht und Compliance**: DSGVO (AV-Verträge, Löschfristen, Auskunft), GoBD für Honorarrechnungen, Aufbewahrungsfristen, Hosting in der EU.
9. **KI-Assistenz** (optional, zweite Stufe): Klassifikation und Extraktion eingehender Mails und PDFs, Entwurf von Antworten und Gutachtentexten aus den Maskenfeldern. Immer mit **Freigabe durch den Sachverständigen**, weil die Haftung für das Gutachten persönlich beim SV liegt.

## 2a. Konsequenzen aus dem Spartenmix

- **Zwei Geschäftsarten:** Gebäude, Hausrat und Inhalt sind Massengeschäft mit kurzen Laufzeiten. Hier zählen Durchsatz, Vorlagen und mobile Ortstermine. Produkthaftpflicht, Transport, Marine und BU laufen Monate bis Jahre, haben viele Parteien, gesetzliche Fristen und Rechenmodelle. Hier zählen Fristenkontrolle, Beteiligtenverwaltung und Nachvollziehbarkeit.
- **Fristen sind das Haftungsrisiko des Büros.** Die Fristlogik im Prototyp kennt keine Hemmung, keine Unterbrechung und keine Feiertage. Im Produktivsystem braucht sie eine fachliche Abnahme durch einen Juristen und einen Feiertagskalender.
- **Mail-Erkennung:** Die Stichwortbewertung trifft die Sparte bei eindeutigen Mails. Bei Mischfällen (Brand mit Inhalt und BU) muss sie vorschlagen, nicht entscheiden. Das Mittel der Wahl wäre ein Sprachmodell zur Extraktion, mit Bestätigung durch den Menschen.
- **PDF:** Die Standardschrift von jsPDF stellt nur westeuropäische Zeichen dar (kein „ń“, „ł“ usw.). Für Transport und Marine mit osteuropäischen Beteiligten ist eine eingebettete Unicode-Schrift Pflicht.
- **`.msg`:** Outlook speichert Mails standardmäßig als `.msg`. Der Prototyp liest nur `.eml`. Produktiv löst das die Anbindung über Microsoft Graph.

## 3. Zielarchitektur (Vorschlag)

```
Mobile App (Ortstermin, offline, Fotos)     Web-App (Büro)
                 \                          /
                  API (REST/GraphQL, Auth, Rollen)
                  |-- Akten / Workflows / Aufgaben-Engine (Regeln + Ereignisse)
                  |-- Dokumente & PDF-Service (Vorlagen)
                  |-- Mail-Connector (Graph/IMAP) -> Klassifikation -> Zuordnung -> Aufgabe
                  |-- Kalender-, Routing-, Telefonie-Connector
                  |-- Abrechnung (GoBD) / Export DATEV
                  PostgreSQL + Objektspeicher (S3-kompatibel, EU)
```

Die Aufgaben-Engine ist **ereignisgesteuert**. Ereignisse sind z. B. „Mail zugeordnet“, „Termin angelegt“ oder „Frist erreicht“. Jede Regel erzeugt idempotent eine Aufgabe mit stabilem Schlüssel. Genau so arbeitet `aufgaben()` im Prototyp, dort aber nur im Browser.

## 4. Empfohlene Reihenfolge

1. 5 bis 10 echte, anonymisierte Akten aus dem Alltag gegen den Prototyp durchspielen und Lücken notieren.
2. Marktprüfung: Es gibt etablierte Branchenlösungen für Sachverständige. Für jede muss klar sein, warum sie **nicht** reicht, bevor eine Eigenentwicklung beginnt.
3. MVP mit Backend: Akten, Workflows, Aufgaben-Engine, M365-Mail, PDF. Telefonie, Routing und KI kommen später.

## Starten

`index.html` im Browser öffnen. Für die PDF-Erzeugung ist Internet nötig, weil jsPDF über ein CDN geladen wird. Mit „Demo zurücksetzen“ werden die Beispieldaten neu geladen.
