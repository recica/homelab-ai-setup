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

## Live-Verifikation

Echte Command-Outputs vom laufenden Server, kein synthetisches Beispiel.

### `hostnamectl`

```
 Static hostname: homelab
Operating System: Ubuntu 26.04 LTS
          Kernel: Linux 7.0.0-28-generic
    Architecture: x86-64
 Hardware Vendor: GMKtec
  Hardware Model: NucBox_EVO-X2
Firmware Version: EVO-X2 1.09
```

### `docker run hello-world`

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### `ollama list`

```
NAME                      ID              SIZE      MODIFIED
dolphin-mixtral:latest    4f76c28c0414    26 GB     42 minutes ago
qwen2.5:32b               9f13ba1299af    19 GB     9 days ago
llama3.2:latest           a80c4f17acd5    2.0 GB    9 days ago
```

### `free -h` und `ollama ps` während aktiver Inferenz

```
$ free -h
               total        used        free      shared  buff/cache   available
Mem:            61Gi        4.4Gi        24Gi        83Mi        33Gi        57Gi
Swap:           8.0Gi           0B       8.0Gi

$ ollama ps
NAME                      ID              SIZE     PROCESSOR    CONTEXT    UNTIL
dolphin-mixtral:latest    4f76c28c0414    31 GB    100% GPU     32768      4 minutes from now
```

Der hohe `buff/cache`-Wert während der Inferenz zeigt, wie die Unified-Memory-Architektur funktioniert: GPU-Speicherzugriffe der iGPU laufen über denselben physischen RAM und tauchen im System als Cache auf, statt als separates, unsichtbares VRAM.
