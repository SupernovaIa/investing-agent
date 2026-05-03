# 01 — Arquitectura

## Vista de alto nivel

El sistema se compone de cinco capas independientes que se comunican mediante
una base de datos compartida y un planificador de tareas.

```
┌─────────────────────────────────────────────────────────────────┐
│                     CAPA 1 - INGESTA DE DATOS                    │
│                                                                  │
│  yfinance / FMP / Alpha Vantage / FRED / News APIs              │
│           │                                                      │
│           ▼                                                      │
│   asset_snapshots (precio, ATH, drawdown, fundamentales)         │
│   asset_news (titulares, summaries, sentiment)                   │
│   events (earnings, dividendos, M&A, eventos macro)              │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CAPA 2 - ESTADO DEL USUARIO                  │
│                                                                  │
│  users · strategies · positions · theses · alerts · watchlist    │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CAPA 3 - MOTOR DE REGLAS                     │
│                                                                  │
│  Evalúa cada (activo, usuario) contra la strategy:               │
│    · cruces de umbral                                            │
│    · cumplimiento de criterios de entrada/salida                 │
│    · earnings próximos                                           │
│    · candidatos de rotación                                      │
│                                                                  │
│  Output: lista de triggers/eventos por usuario                   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CAPA 4 - GENERADOR DE INFORMES (LLM)         │
│                                                                  │
│  Para cada (activo, usuario) ensambla:                           │
│    · datos objetivos del snapshot más reciente                   │
│    · noticias relevantes recientes                               │
│    · estado vs criterios de la strategy                          │
│    · tesis declarada por el usuario                              │
│    · histórico de informes anteriores (deltas)                   │
│                                                                  │
│  Llama a Claude con prompt versionado → informe estructurado     │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CAPA 5 - ENTREGA                             │
│                                                                  │
│  Dashboard web · Telegram · Email · Notion sync                  │
└─────────────────────────────────────────────────────────────────┘
```

## Cadencias de ejecución

| Tarea                              | Frecuencia          | Capa  |
|------------------------------------|---------------------|-------|
| Pull precios y drawdowns           | Diario (mercado)    | 1     |
| Pull noticias                      | Diario              | 1     |
| Pull fundamentales                 | Trimestral + eventos| 1     |
| Pull calendario earnings           | Semanal             | 1     |
| Evaluación de triggers             | Diario              | 3     |
| Informe semanal por activo         | Semanal             | 4     |
| Informe semanal de cartera         | Semanal             | 4     |
| Informe ad-hoc por activo          | Bajo demanda        | 4     |
| Informe disparado por trigger      | Reactivo            | 4     |
| Notificaciones                     | Reactivo            | 5     |

## Decisiones arquitectónicas clave

Las decisiones siguientes están documentadas como ADRs:

- [ADR-001](decisions/adr-001-multi-tenant-from-day-one.md) — Multi-tenant desde el primer día
- [ADR-002](decisions/adr-002-strategy-as-jsonb.md) — Strategy persistida como JSONB
- [ADR-003](decisions/adr-003-llm-as-decision-support.md) — LLM como soporte de decisión, no decisor

## Stack técnico (propuesta inicial)

| Componente                 | Elección                          | Justificación |
|----------------------------|-----------------------------------|---------------|
| Lenguaje                   | Python 3.11+                      | Ecosistema financiero, SDKs LLM |
| ORM                        | SQLAlchemy 2.x                    | Estándar maduro en Python |
| Base de datos              | SQLite → Postgres                 | Empezar simple, migrar si escala |
| Validación                 | Pydantic v2                       | Schemas + type safety |
| Scheduler                  | APScheduler / cron                | MVP simple |
| LLM                        | Anthropic Claude (API)            | Coherencia con estilo de razonamiento |
| Datos de mercado (MVP)     | yfinance                          | Gratis, suficiente para empezar |
| Datos de mercado (futuro)  | Financial Modeling Prep / Alpha   | Métricas avanzadas |
| Notificaciones MVP         | Telegram bot                      | Push instantáneo, gratis |
| Dashboard MVP              | Streamlit                         | Rápido en Python puro |

Esta tabla es indicativa. Cada elección tendrá su propio ADR cuando se confirme.

## Principios técnicos

1. **Idempotencia.** Cualquier tarea de ingesta o generación se puede reejecutar
   sin efectos secundarios.
2. **Deterministic where possible.** El LLM solo entra donde aporta valor real
   (narrativa, razonamiento sobre encaje). Los umbrales y métricas se calculan
   determinísticamente.
3. **Separación de código y configuración.** Las strategies son YAML, los prompts
   son `.md`, los schemas son JSON Schema. Nada de hardcodear.
4. **Trazabilidad por defecto.** Todo informe almacena qué prompt versión usó,
   qué snapshot, qué inputs.
5. **Failure-friendly.** Si yfinance falla, hay fallback a otra fuente. Si el
   LLM falla, el informe se reintenta. Sin pérdida silenciosa de datos.
