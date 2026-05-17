# Docker Task Manager

## Was ich gemacht habe

Für dieses Projekt habe ich eine kleine Task-Manager-Applikation mit Docker Compose gemacht.

Mit der Applikation kann der Benutzer:

- alle Tasks sehen
- einen neuen Task hinzufügen
- einen Task als erledigt markieren
- einen Task löschen

Das Projekt hat drei Haupt-Services:

- Frontend
- Backend
- Datenbank

Das Ziel war, eine kleine Applikation zu machen, bei der die Services wirklich zusammenarbeiten und nicht nur getrennt laufen.

## Warum ich dieses Projekt gewählt habe

Ich habe einen Task Manager gewählt, weil er einfach ist, aber trotzdem die wichtigen Teile einer Multi-Service-Applikation zeigt.

Das Frontend sendet Requests an das Backend.  
Das Backend sendet SQL-Abfragen an die Datenbank.  
Die Datenbank speichert die Tasks.

So hat jeder Service einen echten Zweck.

## Architektur

```text
+----------------+        HTTP        +----------------+        SQL        +----------------+
| Frontend       | -----------------> | Backend        | ---------------> | PostgreSQL     |
| React + Nginx  |                    | Node + Express |                  | Datenbank      |
+----------------+                    +----------------+                  +----------------+
```
## Services
### Frontend

Das Frontend wurde mit React und Vite gemacht.

Es ist der Teil, den der Benutzer im Browser sieht.
Der Benutzer kann dort Tasks hinzufügen, erledigen und löschen.

In Docker wird das Frontend mit Nginx ausgeliefert.

Das Frontend ist hier erreichbar:

http://localhost:8080

### Backend

Das Backend wurde mit Node.js und Express gemacht.

Es stellt die API für das Frontend bereit.
Es verbindet sich auch mit der PostgreSQL-Datenbank.

Das Backend hat diese Endpunkte:

GET    /health
GET    /tasks
POST   /tasks
PUT    /tasks/:id
DELETE /tasks/:id

Das Backend ist hier erreichbar:

http://localhost:3001

Der Healthcheck kann hier getestet werden:

http://localhost:3001/health


### Datenbank

Die Datenbank ist PostgreSQL.

Sie speichert die Tasks.
Das Backend erstellt die Tabelle tasks automatisch, wenn es startet.

Die Task-Tabelle hat diese Felder:

id
title
completed
created_at

## Docker Compose

Die Applikation wird mit Docker Compose gestartet.

Es gibt drei Services in docker-compose.yml:

database
backend
frontend

Die Datenbank benutzt das offizielle PostgreSQL-Image.
Das Backend benutzt ein eigenes Dockerfile.
Das Frontend benutzt auch ein eigenes Dockerfile mit einem Multi-Stage-Build.

## Multi-Stage-Build

Das Frontend-Dockerfile benutzt einen Multi-Stage-Build.

Zuerst wird die React-App mit Node.js gebaut.
Danach werden die fertigen Dateien in einen Nginx-Container kopiert.

So ist der fertige Frontend-Container kleiner und sauberer.

## Umgebungsvariablen

Das Projekt benutzt Umgebungsvariablen.

Es gibt eine .env.example Datei im Repository.

Beispiel:

POSTGRES_USER=appuser
POSTGRES_PASSWORD=apppassword
POSTGRES_DB=tasksdb
DATABASE_URL=postgres://appuser:apppassword@database:5432/tasksdb
BACKEND_PORT=3001

Die echte .env Datei wird nicht zu Git hochgeladen, weil sie in .gitignore eingetragen ist.

## Persistente Daten

Die PostgreSQL-Datenbank benutzt ein named volume:

postgres_data

Das bedeutet, dass die Tasks nicht verloren gehen, wenn die Container neu gestartet werden.

Zum Beispiel nach diesem Befehl:

docker compose down
docker compose up --build

sind die gespeicherten Tasks immer noch da.

## Healthcheck

Der Datenbank-Service hat einen Healthcheck.

Er benutzt pg_isready, um zu prüfen, ob PostgreSQL bereit ist.

Das Backend wartet, bis die Datenbank healthy ist, bevor es startet.

Das wird so gemacht:

depends_on:
  database:
    condition: service_healthy

Das hilft, Fehler beim Start zu vermeiden.

## Netzwerk

Die Services benutzen ein eigenes Docker-Netzwerk:

app_network

Dadurch können die Services innerhalb von Docker miteinander kommunizieren.

Zum Beispiel kann das Backend die Datenbank mit dem Service-Namen erreichen:

database

## Restart Policy

Alle Services benutzen:

restart: unless-stopped

Das bedeutet, dass Docker versucht, einen Service neu zu starten, wenn er abstürzt.

Das macht die Applikation stabiler.

## Logs

Das Backend schreibt strukturierte Logs nach stdout.

Beispiel-Log:

{
  "timestamp": "2026-05-17T14:05:52.802Z",
  "level": "info",
  "event": "server_started",
  "message": "Backend running on port 3000"
}

Die Logs können mit diesem Befehl angeschaut werden:

docker compose logs backend

## Setup-Anleitung

Zuerst die Beispiel-Umgebungsdatei kopieren:

Copy-Item .env.example .env

Dann das Projekt starten:

docker compose up --build

Das Frontend im Browser öffnen:

http://localhost:8080

Den Backend-Healthcheck testen:

http://localhost:3001/health

## Wie man die Applikation testet

Das Frontend öffnen und diese Aktionen testen:

Einen neuen Task hinzufügen
Einen Task als erledigt markieren
Einen Task löschen
Die Container neu starten
Prüfen, ob die Task-Daten immer noch gespeichert sind

Das Backend kann auch direkt getestet werden:

Invoke-RestMethod -Uri "http://localhost:3001/tasks" -Method GET

Einen Task über PowerShell hinzufügen:

Invoke-RestMethod -Uri "http://localhost:3001/tasks" -Method POST -ContentType "application/json" -Body '{"title":"Test task"}'

## Wichtige Code- und Config-Teile
### Datenbank-Healthcheck

healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
  interval: 5s
  timeout: 5s
  retries: 5

### Named Volume
volumes:
  postgres_data:

### Eigenes Netzwerk
networks:
  app_network:

### Restart Policy
restart: unless-stopped

## Technologie-Entscheidungen

Ich habe React/Vite für das Frontend benutzt, weil man damit einfach ein kleines Frontend machen kann.

Ich habe Node.js und Express für das Backend benutzt, weil es einfach ist und gut für REST APIs passt.

Ich habe PostgreSQL benutzt, weil es eine bekannte relationale Datenbank ist und gut mit Docker funktioniert.

Ich habe Docker Compose benutzt, weil das Projekt mehr als einen Service hat und diese Services zusammen laufen müssen.

## AI-Nutzung

Ich habe AI-Tools während diesem Projekt benutzt.

AI hat mir geholfen bei:

der Planung der Struktur
dem Schreiben von Beispielcode
dem Debugging von Docker- und PowerShell-Problemen
der Verbesserung der README-Datei

Ich habe den Code geprüft und angepasst, wo es nötig war.
Ich verstehe den Grundzweck von Frontend, Backend, Datenbank, Docker Compose Datei, Volume, Netzwerk und Healthcheck.

## Reflexion

Ich habe gelernt, wie mehrere Services mit Docker Compose zusammenarbeiten können.

Vor diesem Projekt habe ich meistens nur an einen einzelnen Container gedacht.
Jetzt verstehe ich besser, wie ein Frontend, Backend und eine Datenbank miteinander kommunizieren können.

Ich habe auch gelernt, dass die Start-Reihenfolge wichtig ist.
Das Backend sollte nicht starten, bevor die Datenbank bereit ist.

Ein anderer wichtiger Punkt war Persistenz.
Ohne Docker Volume würden die Daten der Datenbank verloren gehen.

Wenn ich mehr Zeit hätte, würde ich das Design vom Frontend verbessern.
Ich würde auch bessere Fehlermeldungen und vielleicht ein Benutzer-Login hinzufügen.