# Lokale_Chatbot
A local chatbot solution for IT support provides companies

It is a local chatbot for IT support that provides companies with an efficient and privacy-friendly way to automate internal requests. By using AI directly on your own infrastructure, sensitive data remains completely under control and does not leave the company network. Employees receive quick answers to common IT questions around the clock, which significantly reduces the burden on support teams. 
At the same time, recurring problems can be recorded and analyzed in a structured way in order to derive long-term improvements. Especially in regulated industries, an on-premise solution is a decisive advantage over cloud-based alternatives. 
Modern local chatbots can also be seamlessly integrated into existing systems such as ticket systems or knowledge databases. This not only increases efficiency, but also employee satisfaction. Such a solution is an important step towards intelligent, autonomous IT support processes.

# IT-Support Chatbot — On-Premise RAG Solution

Lokale, DSGVO-konforme Chatbot-Lösung für den IT-Support (L1-Automatisierung) auf Basis von
**Retrieval-Augmented Generation (RAG)**. Läuft vollständig on-premise — keine Daten verlassen
die eigene Infrastruktur.

## Architektur

```
Nutzer → Web-UI (Chat) → Auth (Keycloak/OIDC) → Backend API (FastAPI)
                                                        │
                                                        ▼
                                                  RAG-Service
                                  ┌──────────────┬──────────────┬──────────────┐
                                  │  Retrieval   │ Augmentation │  Generation  │
                                  │ (Vektor-DB)  │ (Prompt-Bau) │   (Ollama)   │
                                  └──────┬───────┴──────────────┴──────┬───────┘
                                         ▼                              ▼
                                    ChromaDB                      Llama 3 (8B)
                                         ▲
                                         │
                              Ingestion-Pipeline (PDFs, Wikis, verschlüsselt)
```

Details siehe [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## Projektstruktur

```
it-support-chatbot/
├── common/             # Geteilte Module: Config, Ollama-Client, PII-Erkennung
├── backend/            # FastAPI Gateway: Auth (RBAC), Endpunkte, Audit-Log
├── rag/                # RAG-Kern: Retriever, Prompt-Builder, Generator, I/O-Filter
├── ingestion/          # Dokumentenaufnahme: Loader, Chunker, Indexer
├── frontend/            # Chat-UI (Vanilla JS + Keycloak-JS)
├── infra/
│   ├── k8s/             # Kubernetes-Manifeste
│   └── keycloak/        # Realm-Export (Rollen, Clients)
└── docs/                 # Architektur- & Betriebsdokumentation
```

## Schnellstart (lokal, Docker Compose)

```bash
# 1. Environment-Variablen setzen
cp .env.example .env

# 2. Stack starten (Keycloak, Ollama, ChromaDB, Backend, Frontend)
docker compose up -d --build

# 3. LLM- und Embedding-Modell in Ollama laden
docker compose exec ollama ollama pull llama3:8b
docker compose exec ollama ollama pull nomic-embed-text

# 4. Wissensbasis befüllen (Dokumente liegen in ./data/source)
docker compose exec backend python -m ingestion.run_ingestion --source-dir /data/source

# 5. Chat-UI öffnen
open http://localhost:8080
```

Keycloak-Admin-Konsole: http://localhost:8081 (Realm-Import: `infra/keycloak/realm-export.json`)

## Sicherheitsmechanismen (Auszug)

| Risiko (siehe Deck, Folie 7) | Maßnahme | Implementiert in |
|---|---|---|
| Halluzinationen | Antwort nur aus Kontext, Quellenangabe, Fallback bei fehlendem Kontext | `rag/prompt_builder.py`, `rag/io_filter.py` |
| Datenabfluss | Air-gapped Deployment, RBAC, PII-Maskierung | `backend/auth_guard.py`, `common/pii.py` |
| Prompt-Injection | Input-Heuristiken, System-Prompt-Hardening | `rag/io_filter.py` |
| Fehlende Nachvollziehbarkeit | Audit-Logging jeder Interaktion | `backend/audit_log.py` |

## Tech-Stack

- **Backend / RAG:** Python 3.11, FastAPI, httpx (async)
- **Vektor-DB:** ChromaDB (embedded/Server-Modus)
- **LLM:** Ollama, `llama3:8b` (Generierung), `nomic-embed-text` (Embeddings)
- **Auth:** Keycloak (OIDC, RBAC über Realm-Rollen)
- **Frontend:** Vanilla HTML/CSS/JS + `keycloak-js`
- **Deployment:** Docker Compose (lokal) / Kubernetes (produktiv, siehe `infra/k8s`)

## Status

Funktionsfähiges Grundgerüst (PoC-Stufe gemäß Phasenplan, Folie 6/7: *Discovery → Data Prep → PoC & RAG*).
Nächste Schritte: Integration ITSM/AD (Phase 4), Pilot mit IT-Team (Phase 5), Rollout (Phase 6).
