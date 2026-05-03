# ADR-002: Strategy persistida como JSONB

## Estado

Aceptada — 2026-05-03

## Contexto

Una strategy es un objeto complejo y heterogéneo entre usuarios:

- Un usuario contrarian/growth tiene sub-estrategias con criterios de drawdown
- Un dividend investor tiene criterios de yield mínimo y payout ratio
- Un bogleheads-style tiene solo aportaciones periódicas a ETFs
- Un crypto-maxi tiene allocation entre cadenas y triggers diferentes

Modelar esto en tablas relacionales rígidas implica:

- Muchas tablas con muchas columnas opcionales
- Joins complejos para reconstruir una strategy
- Cambios en el schema cada vez que aparece un nuevo tipo de criterio
- Imposibilidad de modelar criterios libres del usuario

## Decisión

La strategy se persiste como **un campo `JSONB` (o `JSON` en SQLite) en la
tabla `strategies`**, validado contra `schemas/strategy.schema.json`.

```sql
CREATE TABLE strategies (
    id UUID PRIMARY KEY,
    name VARCHAR NOT NULL,
    owner_user_id UUID REFERENCES users(id),
    version INT NOT NULL,
    config JSONB NOT NULL,
    is_active BOOL NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## Consecuencias

**Positivas:**

- Cualquier shape de strategy se soporta sin cambios de schema
- Plantillas pueden compartirse exportando/importando JSON/YAML
- Versionado natural: cada cambio del usuario crea un nuevo registro
- El JSON Schema sirve como documentación ejecutable
- Carga de strategy desde YAML se hace sin transformación intermedia

**Negativas:**

- No hay constraints relacionales sobre el contenido del JSON
- Queries que filtren por contenido del JSON son menos eficientes
  (mitigable con índices GIN en Postgres)
- Hay que mantener disciplina en la validación

**Mitigaciones:**

- Validación obligatoria con Pydantic + JSON Schema antes de cualquier
  inserción/actualización
- Helpers tipados para acceder a campos comunes (no acceso directo al dict)
- Tests automáticos contra strategies de ejemplo

## Alternativas consideradas

- **Tablas relacionales completas:** rechazada por rigidez ante heterogeneidad
  de strategies.
- **Una tabla por tipo de sub-estrategia:** rechazada por complejidad y porque
  obliga a cerrar la lista de tipos prematuramente.
- **EAV (Entity-Attribute-Value):** rechazada por ser un anti-patrón conocido
  en datos estructurados.

## Referencias

- `schemas/strategy.schema.json` — schema formal
- `docs/03-strategy-spec.md` — especificación
- `strategies/javi-v1.yaml` — ejemplo de referencia
