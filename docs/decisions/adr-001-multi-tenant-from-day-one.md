# ADR-001: Multi-tenant desde el primer día

## Estado

Aceptada — 2026-05-03

## Contexto

El proyecto comienza para uso personal de un único usuario, pero con la
intención explícita de abrirse a otros usuarios en el futuro. La estrategia
del usuario inicial está bien definida (ver `strategies/javi-v1.yaml`), y
cualquier otro usuario tendría una distinta.

Existen dos enfoques:

1. **Single-user ahora, refactor a multi-user después:** más simple al principio,
   pero introduce deuda técnica significativa cuando llegue el momento de
   migrar (cambiar todas las queries, añadir scoping, migrar datos).

2. **Multi-tenant desde el día uno:** todas las entidades subjetivas llevan
   `user_id` desde el principio. Pequeño overhead inicial pero sin migración
   futura.

## Decisión

Adoptamos **multi-tenant desde el primer día**.

Todas las entidades que dependen del usuario incluyen `user_id`:

- `positions`, `theses`, `alerts`, `reports` → `user_id` obligatorio
- `strategies` → `owner_user_id` (puede ser NULL si es plantilla pública)
- `assets`, `asset_snapshots`, `asset_news`, `events` → **sin** `user_id`,
  son datos compartidos

El usuario inicial es simplemente "user_id = 1". El sistema, conceptualmente,
ya soporta N usuarios desde el primer commit.

## Consecuencias

**Positivas:**

- Migración futura a multi-user es trivial (no hay refactor)
- Fuerza a separar claramente datos objetivos (compartidos) de subjetivos
  (por usuario), lo cual mejora el diseño general
- Permite tener desde el principio plantillas de strategy compartibles
- Permite testar con múltiples usuarios ficticios desde el día uno
  (útil para validar que la strategy realmente es intercambiable)

**Negativas:**

- Pequeño overhead en cada query: `WHERE user_id = ?`
- Más cuidado al definir índices

**Mitigaciones:**

- Helper de "current user context" para no pasar `user_id` manualmente en
  cada llamada
- Tests automatizados que verifican aislamiento entre usuarios

## Alternativas consideradas

- **Single-user con migración posterior:** rechazada por coste futuro de
  refactorización y por el riesgo de acumular suposiciones single-user que
  son difíciles de detectar tarde.

## Referencias

- `docs/02-data-model.md` — implementación concreta
