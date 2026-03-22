---
title: "Konzept für den Betrieb von EthosAI"
brief: "Beschreibung des operativen Lebenszyklus: prioritätsbasierte Startprüfungen, Task-Dispatching, Logging und Behandlung von Priorität-1-Aufgaben zur Laufzeit."
status: final
version: "0.2.0"
author: Robert Alexander Massinger
date: 2024-09-30
history:
  - date: 2024-09-30
    change: "Erstversion des Betriebskonzepts."
  - date: 2026-02-14
    change: "Front-Matter hinzugefügt; Status unverändert."
tags: [betrieb, lebenszyklus, startup, task-dispatching]
code-release-versions:
  - "0.2.0"
  - "1.1.0"
implemented-features:
  - "Prioritätsbasierte Start-Checkliste"
  - "Task-Dispatching im ProcessModel"
  - "Phase State Machine (OFF→STARTING→RUNNING→STOPPING)"
  - "State Persistence (JSON + Backups)"
fulfillment: "90%"
fulfillment-note: "deinitialize() hat TODO: persist state at shutdown."
---
### Konzept für den Betrieb von EthosAI

#### 1. **EthosAI.start() - Initialisierung und Checkliste**
Beim Start des EthosAI-Systems wird eine automatische Checkliste durchlaufen, um den Zustand und notwendige Aufgaben zu überprüfen:

1. **Prüfung auf Prio 1 Aufgaben:**
   - Überprüfe, ob es Prio 1 Updates gibt.
   - Überprüfe, ob es Prio 1 Service Requests gibt.
   - Überprüfe, ob es andere Prio 1 Aufgaben gibt.

2. **Prüfung auf Prio 2 Aufgaben:**
   - Überprüfe, ob es Prio 2 Updates gibt.
   - Überprüfe, ob es Prio 2 Service Requests gibt.
   - Überprüfe, ob es andere Prio 2 Aufgaben gibt.

3. **Prüfung auf niedere Prioritätsaufgaben:**
   - Überprüfe, ob es niedere Prioritätsaufgaben gibt (z.B. Lernaufgaben, neue Situationen).

4. **Protokollierung:**
   - Für jeden dieser Schritte wird ein Protokoll erstellt, das im „Protocols“-Ordner abgelegt wird.

5. **Start des normalen Betriebs:**
   - Wenn keine Prio 1 Aufgaben anstehen, wird der normale Betrieb gestartet, indem die `ProcessModule.execute` Methode aufgerufen wird.

6. **Logdateien:**
   - Alle Schritte werden in einer Logdatei protokolliert, um eine Nachverfolgbarkeit zu gewährleisten.

#### 2. **Prio 1 Aufgaben während des Betriebs**
Falls während des normalen Betriebs Prio 1 Aufgaben eintreffen:

1. **Warnmeldung:**
   - EthosAI Life gibt eine sofortige Meldung aus.

2. **Sicherheitsmodus:**
   - Das System (EthosAI) begibt sich in eine sichere Position, die den aktuellen Betrieb unterbricht.

3. **Unterbrechung des Betriebs:**
   - Der normale Betrieb wird gestoppt.

4. **Bearbeitung der Prio 1 Aufgaben:**
   - Die Prio 1 Aufgaben werden sequentiell ausgeführt.

5. **Neustart des Betriebs:**
   - Nach Abschluss der Prio 1 Aufgaben wird der normale Betrieb neu gestartet.

#### 3. **Prio 2 Aufgaben während des Betriebs**
Falls Prio 2 Aufgaben während des normalen Betriebs auftreten:

1. **Informationsmeldung:**
   - EthosAI Life informiert die Umgebung über bevorstehende Tätigkeiten.

2. **Vorbereitung auf eine Ruhephase:**
   - Das System (EthosAI) sucht eine geeignete Ruhephase, um die Aufgaben zu erledigen.

3. **Unterbrechung des Betriebs:**
   - Der normale Betrieb wird in der Ruhephase gestoppt.

4. **Bearbeitung der Prio 2 Aufgaben:**
   - Die Prio 2 Aufgaben werden sequentiell abgearbeitet.

5. **Neustart des Betriebs:**
   - Nach Abschluss der Prio 2 Aufgaben wird der normale Betrieb neu gestartet.

#### 4. **Niedrige Prioritätsaufgaben während des Betriebs**
Bei Aufgaben mit niedriger Priorität:

1. **Selbstständige Entscheidung:**
   - EthosAI Life entscheidet eigenständig, ob und wann diese Aufgaben ausgeführt werden, solange keine Gefahr für andere besteht.

2. **Rückkehr zum normalen Betrieb:**
   - Nach Abschluss der Aufgaben kehrt EthosAI Life ggf. zum normalen Betrieb zurück.

#### 5. **EthosAI.stop() - Abschaltprotokoll**
Beim Stoppen des Systems:

1. **Protokollierung des Shutdowns:**
   - Jeder Shutdown wird protokolliert (z.B. „stoppe normalen Betrieb“).

2. **Prioritätsbasierter Shutdown:**
   - Wenn eine Prio 1 Priorität beim Shutdown übergeben wird, können Sicherheitsprotokolle übergangen werden.
   - Bei allen anderen Prioritäten:
     - Ausgabe einer Information an die Umgebung.
     - Suchen einer Position, die für alle sicher ist, bevor der Shutdown vollzogen wird.

#### 6. **Zusammenfassung**
Dieses Konzept gewährleistet, dass EthosAI Life in verschiedenen Situationen auf eine strukturierte und prioritätsbasierte Weise handelt, dabei stets die Sicherheit und Effizienz im Auge behält und alle Prozesse lückenlos dokumentiert.