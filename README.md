# Homelab AI Setup

Dokumentation meines Homelab-Servers für lokale KI-Workloads (Ollama) und
Identity Management (Keycloak).

## Hardware

| Komponente | Spezifikation |
|---|---|
| Gerät | GMKtec NucBox EVO-X2 |
| CPU | AMD Ryzen AI MAX+ 395 w/ Radeon 8060S |
| RAM | 128 GB physisch — davon reserviert das BIOS ~64 GB als dediziertes GPU-VRAM (Unified Memory Architecture), der Rest (~61 GB) ist normaler System-RAM |
| Storage | ~1.8 TB SSD |

## Software / Netzwerk

- **OS:** Ubuntu Server 26.04 LTS
- **IP:** statisch, `192.168.0.36`
- **Orchestrierung:** k3s (single-node Kubernetes)
- **LLM-Runtime:** [Ollama](https://ollama.com), GPU-beschleunigt über ROCm (siehe unten)
- **Identity Provider:** [Keycloak](https://www.keycloak.org) 26.4 — siehe [homelab-keycloak](https://github.com/recica/homelab-keycloak)

## GPU-Beschleunigung (ROCm)

Die iGPU (Radeon 8060S) lief anfangs ungenutzt mit — Ollama fiel auf CPU-Inferenz zurück, obwohl `/dev/kfd` und `/dev/dri` vorhanden waren. Drei Dinge waren zusammen nötig, siehe `manifests/ollama-deployment.yaml`:

1. `ollama/ollama:rocm`-Image statt `latest`
2. `OLLAMA_IGPU_ENABLE=1` — iGPUs sind bei Ollama standardmäßig deaktiviert
3. `securityContext.privileged: true` — ein unprivilegierter Pod bekam `EPERM` beim Lesen der GPU-Topologie unter `/sys/class/kfd`, trotz korrekter Datei-Rechte (Capability-Check, kein DAC-Problem)

Ergebnis: `ollama ps` zeigt `100% GPU`, ROCm erkennt `64.0 GiB` VRAM.

Gemessen mit `dolphin-mixtral` (26 GB, MoE): **29.5 Tokens/s** Generierung, **128.7 Tokens/s** Prompt-Verarbeitung — auf einer iGPU ein flüssig nutzbarer Wert, weit weg von der vorherigen CPU-only-Geschwindigkeit.

## Installierte Ollama-Modelle

| Modell | Größe | Zweck |
|---|---|---|
| qwen2.5:32b | 19 GB | Allzweck, stark bei technischen Themen |
| llama3.2:latest | 2.0 GB | Schnelle, kleine Anfragen |
| dolphin-mixtral | 26 GB | Weniger restriktives Alignment für Grenzthemen (z.B. Security-Recherche), 29.5 Tok/s auf der GPU |

## Keycloak

Läuft nicht mehr hier als Dev-Mode-Docker-Container, sondern produktiv auf dem k3s-Cluster (Postgres-Backend, Traefik-Ingress, echte Realm-Konfiguration). Details, Manifeste und Setup-Doku: [homelab-keycloak](https://github.com/recica/homelab-keycloak).

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
