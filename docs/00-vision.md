# 00 — Visión

## Qué problema resuelve

Un inversor particular con una estrategia bien definida (criterios de entrada,
salida, sizing, reservas, playbooks de referencia) gasta mucho tiempo en
trabajo repetitivo:

- Comprobar precios y drawdowns desde máximos
- Recordar fechas de earnings y revisar resultados
- Releer su tesis original cuando un activo se mueve
- Evaluar si una caída cumple sus criterios o no
- Detectar si una noticia rompe la tesis de alguna posición

Ese trabajo es **mecánico cuando los criterios están claros**, pero consume
atención que se podría dedicar a las decisiones realmente importantes
(entrar/salir, reasignar, cambiar la tesis). Además, hacerlo manualmente
introduce ruido emocional: mirar precios cada día tienta a actuar fuera de plan.

## Qué propone el agente

Un sistema que:

1. **Mantiene un informe vivo por activo** que el usuario consulta cuando quiere,
   actualizado al menos semanalmente.
2. **Avisa solo cuando hay algo accionable** (cruce de umbral, earnings próximo,
   cambio relevante en la tesis).
3. **Aplica la estrategia del usuario de forma consistente**, sin sesgos
   emocionales ni días flojos.
4. **Genera análisis con la calidad y el estilo del usuario**, no genéricos.
5. **Mantiene al usuario en el loop de decisión.** El agente no compra ni vende.

## Para quién

**Primer usuario:** un inversor particular con framework definido (ver
[`strategies/javi-v1.yaml`](../strategies/javi-v1.yaml)).

**Usuarios futuros:** cualquier inversor con una estrategia articulable como
conjunto de reglas (value, contrarian, dividend, growth, crypto-heavy, etc.).
La estrategia es intercambiable; el motor del agente, el mismo.

## Principios rectores

1. **Decision support, no decision making.** El agente prepara, el humano decide.
2. **Estrategia como dato, no como código.** El framework vive en YAML/JSON
   versionado, editable sin tocar el código del agente.
3. **Separación de capas objetiva y subjetiva.** Los datos del activo son
   compartidos; la tesis, posición y análisis son por usuario.
4. **Disciplina sobre intuición.** Los umbrales son deterministas; el LLM razona
   sobre el contexto pero no inventa criterios.
5. **Transparencia auditable.** Todo informe tiene trazabilidad: qué datos usó,
   qué reglas aplicó, qué razonamiento hizo.

## Lo que el agente NUNCA hará

- Ejecutar operaciones reales en brokers
- Recomendar productos financieros con conflicto de interés
- Sustituir el juicio del usuario en decisiones materiales
- Garantizar rentabilidades

## Métrica de éxito

El usuario:
- Mira precios menos veces al día
- Toma decisiones consistentes con su estrategia
- No se pierde catalizadores relevantes
- Detecta antes las rupturas de tesis
- Tiene mejor histórico auditable de por qué entró/salió de cada posición
