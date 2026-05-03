# 07 — Roadmap

## Estado actual

✅ **Fase 0 — Diseño y documentación** (en curso)

Documentación inicial completa, schemas formales, strategy de referencia (Javi v1)
escrita en YAML. ADRs documentando decisiones clave.

## Fases

### Fase 1 — Cimientos (Semana 1-2)

**Objetivo:** repo funcional con modelo de datos, strategy cargable y CLI básico.

- [ ] Setup del proyecto Python (pyproject.toml, dependencies, formato)
- [ ] Modelos SQLAlchemy de las entidades del data model
- [ ] Migraciones con Alembic
- [ ] Modelos Pydantic para validar Strategy contra schema
- [ ] Loader de strategy desde YAML
- [ ] CLI básico: `agent strategy load`, `agent user create`, `agent position add`
- [ ] Seed: cargar strategy "Javi v1" + posiciones actuales del usuario

**Entregable:** comando que inicializa la base de datos con el estado actual
del usuario.

### Fase 2 — Ingesta de datos (Semana 3)

**Objetivo:** pipeline diario que mantiene `asset_snapshots` actualizado.

- [ ] Adapter base con interfaz común para fuentes
- [ ] Adapter `yfinance` para precio, ATH, fundamentales básicos, earnings
- [ ] Adapter `FRED` para macro
- [ ] Job diario `agent ingest snapshots`
- [ ] Job semanal `agent ingest events`
- [ ] Cálculo derivado de drawdowns desde snapshots
- [ ] Tests unitarios de los adapters

**Entregable:** snapshot diario que se actualiza solo.

### Fase 3 — Motor de reglas y triggers (Semana 4)

**Objetivo:** evaluación automática de criterios y disparo de alertas.

- [ ] Evaluador de criterios de entrada/salida por sub-estrategia
- [ ] Evaluador de alertas configuradas por el usuario
- [ ] Generador de eventos: earnings T-7, T-1, T+1
- [ ] Notificador Telegram (push de alertas)
- [ ] Tests del motor de reglas con casos golden

**Entregable:** llegan notificaciones cuando hay algo accionable.

### Fase 4 — Generador de informes con LLM (Semana 5-6)

**Objetivo:** informes semanales por activo + informe de cartera.

- [ ] Cliente Anthropic con manejo de errores y reintentos
- [ ] Plantillas de prompts versionadas en `prompts/`
- [ ] Ensamblador de contexto (recolecta inputs y los formatea)
- [ ] Generador `asset-report` con escritura a `reports`
- [ ] Generador `portfolio-report`
- [ ] Cron semanal que regenera todos los informes
- [ ] CLI `agent report generate <ticker>` para ejecución manual
- [ ] Validación de que el output cumple la estructura de `04-report-spec.md`

**Entregable:** informes semanales generados y consultables.

### Fase 5 — Dashboard (Semana 7-8)

**Objetivo:** UI básica para consultar informes y gestionar posiciones.

- [ ] Streamlit app con vista de cartera
- [ ] Detalle de activo con informe completo
- [ ] Edición de tesis con versionado
- [ ] Configuración de alertas
- [ ] Botón "regenerar informe" manual
- [ ] Histórico de informes con diff visual

**Entregable:** dashboard local utilizable a diario.

### Fase 6 — Integraciones de salida (cuando sea necesario)

- [ ] Sync a Notion (informes como páginas)
- [ ] Email digest semanal
- [ ] Mejoras en Telegram (botones interactivos)

### Fase 7 — Calidad y refinamiento

- [ ] Backtest de la strategy contra datos históricos
- [ ] Análisis de calidad de informes (LLM-as-judge)
- [ ] Optimización de costes de LLM
- [ ] Detección automática de ruptura de tesis

### Fase 8 — Multi-usuario y plantillas

- [ ] Onboarding de usuario nuevo desde plantilla
- [ ] Catálogo de strategies plantilla
- [ ] Hosting compartido (si tiene sentido)

## No-objetivos (al menos por ahora)

- Ejecución automática de operaciones
- Asesoramiento financiero regulado
- Conexión directa a brokers
- Optimización de cartera (Markowitz et al.)
- Comparador de productos financieros

## Decisiones que dependen de la experiencia de uso

- ¿Migrar a Postgres o seguir con SQLite?
- ¿Hosting cloud o self-hosted en VPS?
- ¿Pasar de Streamlit a Next.js?
- ¿Abrir a usuarios externos? ¿Como qué (open source / SaaS / mixto)?

Estas se aplazan hasta tener datos reales de uso.
