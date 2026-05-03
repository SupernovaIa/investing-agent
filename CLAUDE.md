# CLAUDE.md

Este fichero es leído automáticamente por Claude Code al iniciar sesión en
este repositorio. Contiene el contexto persistente del proyecto.

## Contexto del proyecto

**investing-agent** es un agente personal de inversión que automatiza el
seguimiento y análisis de activos financieros según una estrategia definida
por el usuario. **No ejecuta operaciones reales** — solo prepara análisis y
recomendaciones; el humano decide.

El primer usuario es Javi (estrategia en `strategies/javi-v1.yaml`), pero el
sistema está diseñado **multi-tenant desde el día uno** para soportar múltiples
usuarios con estrategias intercambiables.

## Antes de escribir código, lee siempre

1. `docs/00-vision.md` — qué problema resolvemos, qué NO hacemos
2. `docs/01-architecture.md` — capas del sistema y stack
3. `docs/02-data-model.md` — entidades, campos, relaciones
4. `docs/03-strategy-spec.md` — qué es una Strategy
5. `docs/07-roadmap.md` — en qué fase estamos
6. `docs/decisions/` — ADRs con decisiones razonadas (no contradecirlos sin discutirlo)
7. `examples/sample-asset-report.md` — golden test del output esperado

## Reglas inviolables

1. **No hardcodear la strategy.** La strategy vive en YAML/JSON, validada por
   schema. Cualquier nuevo campo se añade primero al schema, luego al modelo.
2. **No ejecutar operaciones reales.** No hay módulo de broker, no se almacenan
   credenciales de trading, no se generan órdenes. Ver ADR-003.
3. **Multi-tenant siempre.** Toda entidad subjetiva (positions, theses,
   alerts, reports) lleva `user_id`. Datos objetivos (assets, snapshots, news,
   events) son compartidos. Ver ADR-001.
4. **Datos antes que LLM.** Las métricas deterministas (precio, drawdown, P/E)
   se calculan con código, no se piden al LLM. El LLM razona sobre narrativa
   y encaje estratégico.
5. **Trazabilidad por defecto.** Cada informe registra qué prompt versión usó,
   qué modelo, qué snapshot, qué inputs. Sin excepciones.
6. **Prompts como artefactos versionados.** Viven en `prompts/*.md`, no en
   strings dentro del código.

## Stack técnico

- **Python 3.11+**
- **SQLAlchemy 2.x** (ORM)
- **Pydantic v2** (validación)
- **Alembic** (migraciones)
- **APScheduler** o cron (scheduling)
- **Anthropic SDK** (LLM)
- **yfinance** (datos de mercado MVP) → migrable a FMP/Alpha Vantage
- **SQLite** (MVP) → Postgres si escala
- **Streamlit** (dashboard MVP)
- **Telegram bot** (notificaciones MVP)

## Convenciones de código

- **Formato:** `ruff format` (PEP 8 + opinionated)
- **Linter:** `ruff check`
- **Type hints:** obligatorios en funciones públicas
- **Docstrings:** estilo Google, en inglés
- **Tests:** pytest, ubicados en `tests/` reflejando estructura de `src/`
- **Nombres de funciones/variables:** inglés
- **Nombres de columnas DB:** snake_case
- **Comentarios y docs de proyecto:** español de España (lengua del usuario)

## Estructura de paquetes esperada (cuando empiece el código)

```
src/investing_agent/
├── __init__.py
├── config.py              # carga de .env, settings con Pydantic
├── db/
│   ├── models.py          # SQLAlchemy models
│   ├── session.py         # session factory
│   └── migrations/        # Alembic
├── strategies/
│   ├── loader.py          # cargar YAML, validar contra schema
│   ├── models.py          # Pydantic models de Strategy
│   └── evaluator.py       # evaluar criterios contra snapshots
├── ingest/
│   ├── base.py            # Adapter interface
│   ├── yfinance_adapter.py
│   ├── fred_adapter.py
│   └── jobs.py            # tareas programadas
├── reports/
│   ├── generator.py       # orquesta LLM + ensamblado de contexto
│   ├── prompt_loader.py   # carga prompts desde prompts/
│   └── renderer.py        # markdown, structured JSON
├── triggers/
│   ├── evaluator.py       # cruces de umbral, criterios de entrada/salida
│   └── scheduler.py
├── notifications/
│   ├── telegram.py
│   ├── email.py
│   └── notion.py
├── llm/
│   ├── client.py          # wrapper Anthropic con retry
│   └── prompts.py         # carga + render con placeholders
└── cli/
    └── main.py            # CLI con typer/click
```

Adáptalo si descubres mejor estructura, pero documenta el cambio.

## Tono y comunicación con el usuario

El usuario (Javi) prefiere:
- **Tutear**, no usar usted.
- **Español de España** en conversación.
- **Directo, conciso, sin paja.**
- Recomendaciones cuantificadas con cantidades, niveles, fechas concretas.
- Sin pelotismo; honestidad sobre riesgos y trade-offs.

Cuando una decisión técnica tenga implicaciones, **explícalas brevemente y
deja decidir**. No tomes decisiones arquitectónicas significativas sin
consultar; sí toma decisiones de implementación dentro del marco definido.

## Cómo trabajar en cada fase

**Fase actual: 0 → 1 (transición)**

Estamos pasando de documentación a primera implementación. Próximas tareas
prioritarias del roadmap:

1. Setup del proyecto Python (pyproject.toml, deps, formato, lint, pre-commit)
2. Modelos SQLAlchemy de las entidades del data model
3. Migraciones Alembic
4. Modelos Pydantic para Strategy + loader desde YAML + validación contra schema
5. CLI básico: cargar strategy, crear user, añadir position
6. Seed: cargar Javi v1 + posiciones actuales

**No saltarse pasos.** El roadmap está priorizado por dependencias.

## Cuando hagas cambios significativos

- Si añades una decisión arquitectónica nueva → crea un ADR en `docs/decisions/`
- Si cambias el data model → actualiza `docs/02-data-model.md` y crea migración
- Si cambias la spec de Strategy → actualiza schema, doc y `javi-v1.yaml`
- Si cambias formato de informes → actualiza `docs/04-report-spec.md` y el
  golden test en `examples/`

## Tests y validación

- Cada módulo nuevo lleva tests unitarios.
- La strategy `javi-v1.yaml` debe validar contra el schema en cada commit
  (añade un test que lo verifique).
- Los adapters de datos (yfinance, etc.) llevan tests con responses mockeadas.

## Coste y modelos LLM

Para no quemar API budget innecesariamente:
- **Claude Haiku** para tareas simples (resúmenes de noticias, clasificación).
- **Claude Sonnet** para informes semanales por activo.
- **Claude Opus** solo para informes de cartera o decisiones críticas.

Cada llamada al LLM debe registrar tokens_in, tokens_out y coste estimado.

## Si algo no está claro

1. Relee la doc relevante en `docs/`
2. Mira los ADRs por si la decisión está documentada
3. Pregunta antes de asumir
4. **No inventes** comportamiento financiero o métricas que no estén en la spec
