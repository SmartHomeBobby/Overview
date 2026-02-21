# SmartHomeBobby – Event‑Driven Smart‑Home Orchestrator

SmartHomeBobby ist ein modularer, Event‑getriebener Smart‑Home‑Orchestrator für dein privates Zuhause.  
Ziel ist es, verschiedenste Sensoren, Aktoren, KI‑Dienste (LLMs) und Sicherheitsfunktionen über ein einheitliches, versioniertes System zu verbinden - vollständig lokal betrieben, erweiterbar und gut beobachtbar.

---

## Motivation

Klassische Smart‑Home‑Setups sind oft fragmentiert: Jede Gerätefamilie bringt ihr eigenes Gateway, ihre eigene Cloud und eigene Automationslogik mit.  
HomeBobby löst das, indem alle Komponenten über einen gemeinsamen Event‑Bus gekoppelt werden und ein zentraler Orchestrator auf Basis von Workflows entscheidet, was wann passieren soll (z.B. Licht, Musik, Alarm, Benachrichtigungen).

---

## Kernideen

- **Event‑Driven Architecture**  
  Alles basiert auf Events. Sensoren, Aktoren, LLM‑Module und andere Dienste kommunizieren über einen zentralen Message‑Bus (z.B. MQTT) per Publish/Subscribe. Der Orchestrator wertet Events aus, führt Workflows aus und erzeugt neue Events.

- **Modulare Services**  
  Jedes Modul ist ein eigenständiger Service (z.B. Sensor‑Adapter, Licht‑Steuerung, LLM‑Backend, Notification‑Dienst, Alarm‑Logik), der unabhängig entwickelt, deployt und skaliert werden kann.

- **Git‑basierte Versionierung**  
  Orchestrator, Module und Workflows sind in privaten Git‑Repositories versioniert. Ein Control‑Plane‑Teil des Orchestrators kann neue Versionen aus Git laden und auf die jeweiligen Hosts ausrollen (GitOps‑ähnlicher Ansatz).

- **Lokales Monitoring / Observability**  
  Alle Events und Logs tragen Trace‑IDs bzw. Correlation‑Informationen und können in einen lokalen Elastic‑Stack oder eine ähnliche Observability‑Lösung ingestiert werden, um Abläufe, Fehler und Sicherheitsereignisse nachzuvollziehen

- **Priorisierte Workloads**  
  Kritische Workflows (z.B. Alarmanlage) werden priorisiert behandelt, sowohl auf Topic‑Ebene (eigene Alarm‑Topics) als auch in den Worker‑Pools des Orchestrators.

---

## Architekturüberblick

### Event‑Bus

- Typischerweise **MQTT** als leichtgewichtiges IoT‑Messaging‑Protokoll (Broker‑basiertes Pub/Sub).
- Alle Module verbinden sich mit dem Broker, publizieren Events und subscriben auf relevante Topics.
- Kritische Themen (z.B. `alarm.critical.*`) können gesondert behandelt werden.

### Orchestrator

- Zentrale Instanz, die:
  - auf definierte Topics hört (Sensoren, User‑Events, System‑Events),
  - Workflow‑Definitionen evaluiert (Bedingungen + Aktionen),
  - Folge‑Events erzeugt (Kommandos, Notifications, weitere Workflow‑Schritte).
- Enthält:
  - **Rule/Workflow‑Engine** (z.B. Workflows in C#),
  - **State‑Store**, der aus Events den aktuellen Zustand („Licht an? Alarm scharf?“) ableitet,
  - **Control‑Plane** für Deployments (Kommunikation mit Host‑Agenten, die Git‑Repos clonen/pullen und Services starten).

### Module

Beispiele:

- **Sensor‑Module**  
  Lesen physische/virtuelle Sensoren (Temperatur, Bewegung, Fensterkontakte, Stromverbrauch) und publizieren Events wie `sensor.livingroom.temperature`.

- **Aktor‑Module**  
  Steuern Geräte (Licht, Musik, Heizung) basierend auf `command.*`‑Events und senden Status‑Events zurück.

- **LLM-/Agent‑Module**  
  Verarbeiten `llm.request`‑Events (z.B. Chat, Planungs‑ oder Agenten‑Tasks) und senden `llm.response` zurück.  

- **Notification‑Module**  
  Senden Push‑Nachrichten, E‑Mails, Telegram‑Messages o.ä. auf Basis von `notification.send`.

- **Security-/Alarm‑Module**  
  Evaluieren sicherheitsrelevante Ereignisse, setzen Alarmzustände, steuern Sirenen und Benachrichtigungen.

---

## Workflows & Zustand

### Workflows

- Workflows beschreiben, was auf bestimmte Ereignisse hin passieren soll (z.B. „Wenn Bewegung im Wohnzimmer & Alarm scharf & Nacht, dann löse Alarm aus und sende Notification“).
- Geplant ist die Umsetzung der Workflows in **C#** als separater Workflow‑Service, der über Events mit dem Orchestrator interagiert.

### Zustand (State‑Store)

- Events sind Momentaufnahmen, Workflows brauchen aber **aktuellen Zustand**.
- Ein interner State‑Store im Orchestrator:
  - hält den letzten bekannten Zustand pro Entity (z.B. pro Gerät/Raum),
  - wird aus eingehenden Events fortlaufend aktualisiert,
  - kann für kritische Flags (z.B. Alarm scharf/unscharf) persistent gespeichert werden.

---

## Observability & Elastic‑Stack

- Alle Module loggen strukturierte Events (JSON) mit u.a.:
  - `trace_id`, `span_id`,
  - `module`, `host`, `priority`,
  - Payload‑Details.
- Logs/Events können in einen lokalen **Elastic‑Stack** ingestiert werden, um:
  - End‑to‑End‑Traces einzelner Workflows zu analysieren,
  - Dashboards und Metriken zu erstellen,
  - Sicherheitsereignisse zu überwachen (ähnlich zu Elastic‑SIEM‑Homelab‑Projekten).

---

## Features (geplant/teilweise umgesetzt)

- Event‑getriebene Orchestrierung von Smart‑Home‑Geräten.
- Versionierte Workflows und Konfigurationen in Git.
- C#‑basierte Workflow‑Implementierungen mit typsicheren Event‑DTOs.
- Lokale Observability mit Trace‑IDs und Dashboards (Elastic‑Stack oder Alternative).
- Priorisierte Behandlung sicherheitskritischer Workflows (Alarmanlage).

---

## Roadmap (High‑Level)

1. **MVP**
   - MQTT‑Broker + minimaler C#‑Orchestrator.
   - Dummy‑Sensor‑ und Aktor‑Module.
   - Hardcodierte C# Workflows.
2. **State‑Store & Git‑Flows**
   - Zustandsmodell im Orchestrator,
   - Workflow‑Definitionen in einem eigenen Git‑Repo.
3. **Host‑Agenten & Git‑Deployments**
   - Agent‑Services auf Unraid/Mac mini, die Git‑Repos clonen/pullen und Services starten.
4. **Observability**
   - Strukturierte Logs, Trace‑IDs, lokale Elastic‑ oder Loki/Grafana‑Integration.
5. **Web‑UI „Bobby“**
   - Übersicht über Module, Workflows, Systemstatus, manuelle Szenensteuerung.
6. **LLM-/Agent‑Integration & Alarmanlage**
   - LLM‑Module für Smart‑Home‑Interaktion und Agentic‑Tasks,
   - robuste, priorisierte Alarm‑Workflows.

---

## Status

Projekt ist in aktiver Entwicklung.  
Ziel ist eine robuste, lokal betriebene Smart‑Home‑Plattform, die moderne Event‑Driven‑ und Observability‑Konzepte aus der Cloud‑Welt ins eigene Zuhause bringt.
