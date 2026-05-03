# Documentación

Esta carpeta contiene toda la documentación del proyecto. La numeración
indica orden recomendado de lectura para alguien nuevo.

## Índice

| Documento | Descripción |
|-----------|-------------|
| [`00-vision.md`](00-vision.md)             | Qué problema resuelve, para quién, principios rectores |
| [`01-architecture.md`](01-architecture.md) | Capas del sistema y stack técnico |
| [`02-data-model.md`](02-data-model.md)     | Entidades, campos y relaciones |
| [`03-strategy-spec.md`](03-strategy-spec.md) | Especificación de Strategy como objeto |
| [`04-report-spec.md`](04-report-spec.md)   | Estructura y reglas de los informes |
| [`05-llm-prompts.md`](05-llm-prompts.md)   | Diseño de prompts del LLM |
| [`06-data-sources.md`](06-data-sources.md) | Fuentes de datos de mercado |
| [`07-roadmap.md`](07-roadmap.md)           | Fases del proyecto y estado actual |

## Decisiones arquitectónicas

Las decisiones importantes del proyecto se documentan como **Architecture
Decision Records (ADRs)** en [`decisions/`](decisions/):

- [`adr-001-multi-tenant-from-day-one.md`](decisions/adr-001-multi-tenant-from-day-one.md)
- [`adr-002-strategy-as-jsonb.md`](decisions/adr-002-strategy-as-jsonb.md)
- [`adr-003-llm-as-decision-support.md`](decisions/adr-003-llm-as-decision-support.md)

## Cómo añadir un nuevo ADR

Cuando se tome una decisión arquitectónica significativa:

1. Crea `adr-NNN-titulo-corto.md` con número incremental.
2. Sigue el template: Estado, Contexto, Decisión, Consecuencias, Alternativas.
3. Refiérelo desde el documento principal afectado.

El objetivo es que dentro de 6 meses, cuando dudes "¿por qué hicimos esto así?",
el ADR te lo recuerde.
