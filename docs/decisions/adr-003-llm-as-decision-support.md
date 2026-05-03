# ADR-003: LLM como soporte de decisión, no como decisor

## Estado

Aceptada — 2026-05-03

## Contexto

El sistema podría, técnicamente, conectarse a un broker (Trade Republic, IBKR,
Alpaca, etc.) y ejecutar operaciones automáticamente cuando se cumplen los
criterios de la strategy. Esto eliminaría la fricción del último paso.

Sin embargo, hay tres tipos de riesgo material:

1. **Errores de código:** un bug en el evaluador de reglas puede disparar una
   compra/venta no deseada con consecuencias económicas reales.

2. **Alucinaciones del LLM:** el LLM puede invocar criterios mal interpretados,
   confundir tickers, generar recomendaciones plausibles pero erróneas.

3. **Errores en datos de fuente:** un fallo de yfinance que devuelve un precio
   incorrecto puede activar triggers falsos.

El coste de cualquiera de estos errores es directamente económico para el
usuario.

Por otra parte, el valor diferencial del agente **no está en ahorrar dos clicks
de ejecución**. Está en:

- Mantener la disciplina de la strategy
- Detectar oportunidades y rupturas de tesis a tiempo
- Reducir el tiempo dedicado a mirar precios
- Generar análisis consistentes

Todos estos beneficios se obtienen **sin** ejecutar operaciones automáticamente.

## Decisión

El agente **nunca ejecutará operaciones reales en brokers**. El usuario es
quien aprieta el botón de compra/venta en su broker preferido.

El agente:

- Analiza, recomienda, alerta, recuerda
- Genera informes con recomendaciones cuantificadas
- Mantiene tesis y criterios trazables

El agente NO:

- Se conecta a APIs de broker para ejecutar
- Genera órdenes con tokens de autenticación de operaciones
- Asume autorización implícita por inactividad del usuario

## Consecuencias

**Positivas:**

- El usuario mantiene control y responsabilidad sobre cada operación
- Errores del sistema no tienen consecuencias económicas directas
- Reducción del riesgo regulatorio (no es un robo-advisor regulado)
- Confianza del usuario: el agente nunca actúa por sí mismo

**Negativas:**

- Fricción mínima en la ejecución (clicks en el broker)
- Riesgo de que el usuario no actúe a tiempo en oportunidades

**Mitigaciones:**

- Notificaciones push instantáneas para que el usuario actúe rápido
- Recomendaciones lo suficientemente claras como para ejecutarse en segundos
- Recordatorios escalados si una alerta queda sin atender

## Implicaciones de diseño

- No habrá módulo de "ejecución" en el agente
- No se almacenan credenciales de brokers
- El estado de la cartera se sincroniza por entrada manual del usuario o por
  importación de CSV/screenshot del broker (futuro)

## Alternativas consideradas

- **Ejecución automática con confirmación previa por Telegram:** descartada
  porque añade complejidad y riesgo regulatorio sin beneficio proporcional.
- **Modo "dry-run" con envío de orden lista para copiar:** posible en el
  futuro, no descarta esta decisión.

## Referencias

- `docs/00-vision.md` — principios rectores
