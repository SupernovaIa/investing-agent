# Investing Agent

Agente personal de inversión que automatiza el seguimiento y análisis de
activos financieros de acuerdo con una estrategia definida por el usuario.

## Qué hace

- Mantiene un **informe vivo por activo** en cartera o watchlist, actualizado
  semanalmente o bajo demanda.
- Genera un **informe agregado de cartera** con encaje estratégico global,
  exposición sectorial, candidatos a rotación y calendario de catalizadores.
- Dispara **alertas** cuando un activo cumple criterios de entrada/adición/salida
  según la estrategia del usuario.
- Vigila **catalizadores** (earnings, eventos corporativos, noticias relevantes)
  y avisa con antelación.
- Detecta **rupturas de tesis** comparando hechos nuevos contra la tesis
  declarada por el usuario al añadir el activo.

## Lo que NO hace

- **No ejecuta operaciones automáticamente.** El humano siempre aprieta el botón
  de compra/venta. El agente prepara el análisis y propone; nunca decide solo.
- No es un robo-advisor. No optimiza carteras al estilo Markowitz. No hay
  promesas de rentabilidad.

## Arquitectura conceptual

```
[Datos de mercado]  ─┐
[Calendario eventos]─┼─► [Capa objetiva: snapshots por activo, compartida]
[Noticias]          ─┘                          │
                                                ▼
[Strategy del usuario] + [Posiciones] + [Tesis] ─► [Generador LLM] ─► [Informe + Recomendación]
                                                                            │
                                                                            ▼
                                                              [Notificaciones / Dashboard]
```

Ver [`docs/01-architecture.md`](docs/01-architecture.md) para el detalle.

## Estado del proyecto

🚧 **Fase 0 — diseño y documentación.** El repo contiene actualmente la
especificación, schemas y la estrategia de referencia (Javi v1). El código
funcional empezará a llegar cuando la documentación valide el diseño.

Ver [`docs/07-roadmap.md`](docs/07-roadmap.md) para el plan completo.

## Estructura del repo

- [`docs/`](docs/) — documentación del proyecto (visión, arquitectura, specs).
- [`docs/decisions/`](docs/decisions/) — Architecture Decision Records (ADRs).
- [`strategies/`](strategies/) — estrategias de inversión como YAML versionado.
- [`schemas/`](schemas/) — JSON Schemas para validar Strategy, Thesis y Report.
- [`prompts/`](prompts/) — prompts del LLM como artefactos versionados.
- [`src/`](src/) — código fuente (en construcción).
- [`tests/`](tests/) — tests automatizados (en construcción).
- [`examples/`](examples/) — ejemplos de informes generados.

## Filosofía

1. **Decision support, no decision making.** El agente es un asistente analítico,
   no un trader automático.
2. **Disciplina sobre intuición.** El agente codifica reglas explícitas para
   contrarrestar sesgos emocionales del usuario.
3. **Transparencia por defecto.** Cada recomendación incluye los datos en que
   se basa y los criterios estratégicos que aplica.
4. **Estrategia como dato.** El framework de inversión se define como
   configuración intercambiable, no se hardcodea.
5. **Humano siempre en el loop.** Las acciones reales (comprar, vender) las
   ejecuta el usuario.

## Licencia

[Apache 2.0](LICENSE)
