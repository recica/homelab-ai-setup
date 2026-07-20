# Homelab AI Setup

Dokumentation meines Homelab-Servers für lokale KI-Workloads (Ollama) und
Identity Management (Keycloak).

## Hardware

| Komponente | Spezifikation |
|---|---|
| Gerät | GMKtec NucBox EVO-X2 |
| CPU | AMD Ryzen AI MAX+ 395 |
| RAM | 128 GB |
| Storage | ~1.8 TB SSD |

## Software / Netzwerk

- **OS:** Ubuntu Server 26.04 LTS
- **IP:** statisch, `192.168.0.36`
- **Container-Runtime:** Docker
- **LLM-Runtime:** [Ollama](https://ollama.com)
- **Identity Provider:** [Keycloak](https://www.keycloak.org) 26.4

## Installierte Ollama-Modelle

| Modell | Größe |
|---|---|
| qwen2.5:32b | 19 GB |
| llama3.2:latest | 2.0 GB |

## Keycloak

- Läuft als Docker-Container (`keycloak`, Image `quay.io/keycloak/keycloak:26.4`, Dev-Modus)
- Port: `8080`
- Admin-Zugangsdaten wurden lokal generiert und **nicht** in diesem Repo gespeichert

## Screenshots

Die folgenden Screenshots werden manuell ergänzt.

### 1. `hostnamectl`

_Systeminformationen des Homelab-Servers._

![hostnamectl](screenshots/hostnamectl.png)

### 2. `docker run hello-world`

_Bestätigung, dass Docker korrekt installiert und funktionsfähig ist._

![docker hello-world](screenshots/docker-hello-world.png)

### 3. `ollama list`

_Übersicht der installierten Ollama-Modelle._

![ollama list](screenshots/ollama-list.png)

### 4. `ollama run` mit `free -h` während der Inferenz

_Speicherauslastung während ein Modell aktiv Inferenz durchführt._

![ollama run mit free -h](screenshots/ollama-run-free.png)
