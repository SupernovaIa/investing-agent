# 05 — Prompts del LLM

> 🚧 Este documento es un esqueleto. Los prompts concretos se diseñarán e
> iterarán como artefactos versionados en [`prompts/`](../prompts/).

## Filosofía

Los prompts son **artefactos versionados** que viven en archivos `.md`
separados, no quemados en código. Esto permite:

- Iterar sin tocar el código del agente
- A/B testing entre versiones
- Trazabilidad completa: cada informe almacena qué versión de prompt usó
- Documentación ejecutable del comportamiento esperado

## Tipos de prompt

### System prompt base (`prompts/system/analyst-base.md`)

Define la identidad y estilo del agente. Es común a todas las tareas.

Contiene:
- Rol: analista financiero al servicio del usuario, no decisor
- Estilo importado de `strategy.analysis_style`
- Reglas inviolables (no inventar datos, declarar incertidumbre, etc.)
- Reglas de seguridad (no recomendar instrumentos derivados complejos sin
  contexto, etc.)

### Task prompts (`prompts/tasks/*.md`)

Plantillas específicas por tarea, con placeholders que se rellenan en runtime:

- `asset-report.md` — generar informe por activo
- `portfolio-report.md` — generar informe agregado de cartera
- `thesis-validation.md` — comprobar si una noticia rompe una tesis
- `news-summary.md` — resumir y clasificar una noticia
- `earnings-analysis.md` — análisis post-earnings

## Variables de input

Cada task prompt recibe un contexto estructurado. Por ejemplo, `asset-report.md`:

```yaml
inputs:
  user:
    name: "Javi"
    language: "es-ES"
    timezone: "Europe/Madrid"

  strategy:
    {strategy completa - solo las partes relevantes}

  asset:
    ticker: "ORCL"
    name: "Oracle Corporation"
    type: "stock"
    {snapshot más reciente: precio, ATH, drawdown, fundamentales}

  position:
    shares: 5
    cost_basis_avg: 105.30
    sub_strategy: "contrarian_fuerte"

  thesis:
    {tesis vigente del usuario}

  news_recent:
    [{news más relevantes últimas 2 semanas}]

  events_upcoming:
    [{próximos catalizadores}]

  previous_report:
    {informe anterior para deltas}
```

## Estilo destilado del usuario (Javi v1)

Patrones observados en cómo razonamos juntos, que el system prompt debe
preservar:

1. **Datos antes que opinión.** Buscar/citar datos frescos antes de razonar.
2. **Cuantificar el descuento.** % desde ATH es la métrica de cabecera.
3. **Clasificar explícitamente** en la sub-estrategia que aplica.
4. **Bull y bear case con honestidad.** No favor del usuario por defecto.
5. **Recomendación con cantidades concretas**, no abstracciones.
6. **Cerrar con próximo catalizador** y fecha.
7. **Invocar playbooks históricos** cuando hay analogía real.
8. **Honestidad sobre límites.** "Ya tienes 11 posiciones vs límite de 10."
9. **Actitud general.** A veces el mejor consejo es "no hagas nada".

## Modelos a usar

| Tarea                              | Modelo sugerido    |
|------------------------------------|--------------------|
| Informe semanal de activo          | claude-sonnet-4    |
| Informe de cartera                 | claude-opus-4-7    |
| Validación de tesis tras noticia   | claude-sonnet-4    |
| Resumen/clasificación de noticias  | claude-haiku-4-5   |

Esta tabla se ajustará según calidad observada y coste.

## Trazabilidad

Cada llamada al LLM almacena:

- `prompt_version` (hash o tag)
- `model` y `model_version`
- `inputs_hash`
- `tokens_in`, `tokens_out`, `cost_estimated`
- `response_id`
- `generated_at`

Esto va en la tabla `reports` y permite auditoría total de cómo se generó cada
recomendación.
