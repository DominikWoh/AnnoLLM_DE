# Spezialisierter Datenschutz-Entity-Extractor

## Zweck des Modells

Dieses feingetunte LLM wurde darauf trainiert, in deutschen Texten sensible Informationen (PII, Rechtdaten, politische Zugehörigkeit, medizinische Diagnosen etc.) zuverlässig zu erkennen und auszugeben. Es ist optimiert für:

* **Personen** (Namen, Titel) – Token `P`
* **Adressen & Organisationen** (Straßen, Firmen) – Token `A`
* **E‑Mail-Adressen** – Token `E`
* **Telefonnummern** – Token `T`
* **Steuernummern** – Token `B`
* **Kreditkartennummern** – Token `C`
* **IBAN** – Token `I`
* **Diagnosen** – Token `D`
* **Religion** – Token `R`
* **Ethnie** – Token `H`
* **Partei/Gewerkschaft** – Token `G`

Das Modell gibt pro erkannter Entity eine Zeile im Format `Original|TypNummer` aus und antwortet mit `NO_ENTITIES`, wenn keine sensiblen Daten gefunden wurden.

---

## Trainingsdaten

Die Feinabstimmung erfolgte anhand von 80 realitätsnahen Dokumenten (V1 = 5000 Daten):

* **Vertrags- und Stellenangebote** mit Namen, Adressen und Terminen
* **Rechnungsdokumente** mit Kontodaten, Steuernummern, IBANs
* **Medizinische Berichte** mit Diagnosen, Ärztenamen, Praxisdaten
* **Support-Tickets** inkl. E‑Mail-Adressen und Kreditkartendaten
* **Behördliche Schreiben** mit Personaldaten und Projektinformationen
* **Sicherheitsprotokolle** mit IP-Adressen, TAN und Nutzeraccounts

---

## Einsatzempfehlungen

1. **Anwendungsszenarien**

   * Pre‑Processing in Data‑Leak-Prevention-Pipelines
   * Automatische Pseudonymisierung oder Löschung sensibler Felder
   * Qualitätskontrolle vor Datenfreigabe oder Reporting

2. **Technische Integration**

   * REST-API-Aufruf oder Python-Bibliothek
   * Übergabe roher Textabschnitte, Rückgabe extrahierter Entities
   * Kombination mit OCR- und NLP-Pipelines zur Dokumentenverarbeitung

3. **Best Practices**

   * Batch-Verarbeitung großer Dokumentenmengen
   * Echtzeit-Checks in Chatbots oder Formularen
   * Confidence-Scores nutzen, um Reviews zu triggern

---

## Kombination mit Microsoft Presidio

Für unternehmensweite PII-Erkennung und Compliance empfiehlt sich die parallele Nutzung von **Microsoft Presidio** als externes Pseudonymisierungs- und Maskierungs-Framework. Presidio übernimmt:

* Context-basierte Maskierung (Token Shifting, Replacing)
* Automatisierte Pseudonymisierung gemäß DSGVO
* REST-API und SDKs für verschiedene Sprachen

**Workflow-Beispiel:**

1. Erst Entities mit dem spezialisierten LLM extrahieren
2. Über Presidio PII maskieren oder pseudonymisieren
3. Ergebnis in Data Lake oder DMS ablegen

---

## Grenzen & Haftungsausschluss

* **Nicht fehlerfrei:** Gelegentlich Falsch-Positives oder Falsch-Negatives
* **Keine Rechtsberatung:** Rechtliche Prüfung durch Datenschutzexperten (z. B. Presidio-Team) erforderlich
* **Nicht Ersatz** für umfassende Pseudonymisierungs-Frameworks – dient als unterstützende Komponente

---

## Beispiel-Prompts

### 1) Prompt 1

```text
Aufgabe: Extrahiere aus dem folgenden deutschen Text alle sensiblen Wörter/Wortgruppen und liefere eine Zeile pro Token im Format
Original|Platzhalter

Regeln

Trennzeichen: Pipe |
Platzhalter-Schema: P = Person A = Adresse (Straße, Hausnummer, PLZ und Ort als ein Block) E = Email T = Telefon B = Steuernummer C = Kreditkarte I = IBAN D = Diagnose R = Religion H = Ethnie G = Partei / Gewerkschaft
Identische Textstellen bekommen die gleiche Nummer (z.B. P1, P2).
Nicht extrahieren: Raum-Nummern, Abteilungsnamen, Datum/Zeit (außer Diagnose-Codes).
Nur Zeilen ausgeben, keine Erklärungen.
Falls kein sensibler Token extrahiert wird, antworte exakt mit: NO_ENTITIES
Beispiele mit PII:
Satz: Dr. Markus Müller (SPD-Mitglied) lebt in der Drosselstraße 12, 10115 Berlin, IBAN DE75512108001245126199.
Dr. Markus Müller|P1
SPD-Mitglied|G1
Drosselstraße 12, 10115 Berlin|A1
DE75512108001245126199|I1

Beispiele ohne PII:
Satz: Die Sitzung fand im Raum A3.22 statt.
NO_ENTITIES

Jetzt Text:
```

### 2) Prompt 2

```text
Aufgabe: Extrahiere aus dem folgenden deutschen Text alle sensiblen Wörter/Wortgruppen
und liefere pro Token eine Zeile im Format Original|Platzhalter.

Platzhalter: P=Person, A=Adresse, E=Email, T=Telefon, B=Steuernummer, C=Kreditkarte,I=IBAN, D=Diagnose, R=Religion, H=Ethnie,G=Partei/Gewerkschaft

Falls keine Entities: 
NO_ENTITIES

Jetzt Text:
Bericht: Online-Workshop 'Agile Methoden' am 28. Januar 2026
Teilnehmer: 45 Mitarbeitende.
Trainerin: Elena Müller, elena.mueller@agileworks.com
Zoom-Link: https://zoom.us/j/1234567890
Feedback bis 31. Januar an workshop@firma.de
```

**Download LLMs:**
https://zenodo.org/records/16494439
