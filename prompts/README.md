# Prompts

Los prompts del LLM viven aquí como artefactos versionados, separados del código.

## Estructura

```
prompts/
├── system/
│   └── analyst-base.md       # System prompt común a todas las tareas
├── tasks/
│   ├── asset-report.md       # Generar informe por activo
│   ├── portfolio-report.md   # Generar informe agregado de cartera
│   ├── thesis-validation.md  # Comprobar si una noticia rompe la tesis
│   ├── news-summary.md       # Resumen y clasificación de noticia
│   └── earnings-analysis.md  # Análisis post-earnings
└── README.md                 # Este archivo
```

## Convenciones

- **Plantillas con placeholders**: usamos `{{variable}}` para inyectar contexto
  en runtime.
- **Versionado por nombre**: si hay que iterar drásticamente, se crea
  `asset-report-v2.md` y se decide cuándo migrar.
- **Tests asociados**: cada prompt tiene un set de inputs de ejemplo en
  `examples/` con su output esperado de referencia.

## Variables disponibles típicamente

```
{{user_name}}
{{user_language}}
{{strategy_yaml}}              # strategy completa serializada
{{asset_ticker}}
{{asset_snapshot_json}}        # snapshot más reciente
{{position_json}}              # null si watchlist
{{thesis_json}}                # tesis vigente
{{news_recent_json}}
{{events_upcoming_json}}
{{previous_report_summary}}    # null si es el primer informe
{{current_date}}
```

## Iteración

Cada cambio significativo de prompt:

1. Se crea como variante (ej. `asset-report-v2.md`)
2. Se compara contra ejemplos golden en `examples/`
3. Si la calidad mejora, se promueve a versión activa
4. La versión usada se almacena en cada `report.prompt_version`

## Estado actual

🚧 Los prompts concretos se crearán en la Fase 4 del roadmap. Este directorio
sirve de momento como contrato estructural.
