# Examples

Outputs de referencia que sirven como **golden tests** para validar la
calidad de los informes generados por el agente.

## Para qué sirven

Cuando se programe el generador de informes (Fase 4 del roadmap) y los prompts
se iteren, cada cambio se evalúa contra estos ejemplos para verificar que la
calidad se mantiene o mejora.

Un golden test no es un test automatizado tradicional (no hay match exacto
posible con outputs LLM), sino una **referencia canónica** contra la que
comparar manualmente o mediante LLM-as-judge.

## Ejemplos disponibles

| Archivo | Tipo | Descripción |
|---|---|---|
| `sample-asset-report.md` | Asset report | Informe semanal de Oracle (ORCL) para el usuario Javi. Caso `contrarian_fuerte` con posición ya consolidada. |
| `sample-portfolio-report.md` | Portfolio report | (pendiente) Informe agregado de cartera. |

## Cómo evaluar un nuevo prompt contra un ejemplo

1. **Inputs equivalentes:** preparar un contexto de input (snapshot, position,
   thesis, news) similar al del ejemplo.
2. **Generar:** ejecutar el prompt con el modelo objetivo.
3. **Comparar:** evaluar manualmente o vía LLM-as-judge si el output cumple:
   - Estructura conforme a `docs/04-report-spec.md`
   - Datos cuantificados (drawdown, P&L, niveles concretos)
   - Bull/bear case honestos y con sustancia
   - Recomendación accionable, no abstracta
   - Próximo catalizador con fecha
   - Tono directo, conciso, sin paja
   - Comparación con playbooks si hay analogía real
4. **Registrar:** documentar en qué versión de prompt se generó y si pasó.

## Criterios de aceptación de un nuevo prompt

Un prompt nuevo se considera **regresión inaceptable** si:

- Omite el drawdown desde ATH antes de opinar
- Genera recomendaciones genéricas ("considera X") en vez de cuantificadas
- Inventa datos no presentes en los inputs
- No clasifica el activo en una sub-estrategia explícita
- No incluye bear case real
- No fija próximo catalizador con fecha

Un prompt nuevo se considera **mejora** si:

- Mantiene todos los criterios anteriores
- Reduce tokens de output sin perder información clave
- Detecta mejor los casos límite (saturación de cartera, tesis en riesgo, etc.)
- Invoca playbooks históricos con mayor pertinencia

## Consideración temporal

Los datos de los ejemplos están **anclados a una fecha**. El ejemplo de Oracle
usa datos a 3 mayo 2026. No actualizarlos: el ejemplo es una snapshot canónica,
no un dashboard vivo.

Si la realidad cambia drásticamente y el ejemplo deja de tener sentido (ej:
cambio fundamental en la empresa que invalida la tesis), se reemplaza por un
nuevo ejemplo con activo distinto.
