# 04 — Especificación de informes

Este documento define la estructura, contenido y reglas de generación de los
informes que produce el agente.

> 🚧 Este documento es un esqueleto inicial. Se irá refinando a medida que se
> validen los outputs con casos reales.

## Tipos de informe

1. **Informe por activo** — un activo en cartera o watchlist. Se actualiza
   semanalmente o bajo demanda.
2. **Informe de cartera** — visión agregada de la cartera del usuario.
   Semanal.

## Informe por activo

### Estructura

```markdown
# {TICKER} — {NOMBRE}
{fecha de generación} · {trigger}

## 📌 Snapshot
- Precio: {precio} {moneda}
- ATH: {ath} ({fecha}) · Drawdown: {drawdown_pct}%
- Tu posición: {N acciones} a {coste medio} → P&L: {pnl}
- Estrategia asignada: {sub_strategy}
- Convicción declarada: {1-5}
- Estado de la tesis: 🟢 vigente / 🟡 en riesgo / 🔴 rota

## 🎯 Encaje estratégico hoy
{semáforo} — {explicación de 1-2 frases}
Criterios cumplidos: {lista}
Criterios incumplidos: {lista}

## 📊 Datos clave
{tabla compacta con 5-8 métricas relevantes según el caso}

## 📰 Lo que cambió esta semana
{cambios respecto al informe anterior + noticias relevantes}

## 🐂 Bull case actualizado
{razones a favor con datos frescos}

## 🐻 Bear case honesto
{riesgos reales identificados}

## 📚 Comparación con tu playbook
{referencia a casos análogos en la strategy: "sigue pareciéndose a Meta 2022", etc.}

## 📅 Próximo catalizador
{fecha} · {qué vigilar}

## ✅ Recomendación accionable
{recommendation_type}: {acción específica con cantidades}
{condiciones y niveles para futuras adiciones/salidas}
```

### Reglas de generación

- **Drawdown siempre primero.** Es la métrica de cabecera.
- **Recomendación cuantificada.** No "considera añadir", sino "añade 1 acción a ~$550".
- **Bear case obligatorio.** Si el LLM no encuentra bear case, debe declararlo
  explícitamente como red flag.
- **Catalizador siempre presente.** Si no hay earnings próximo, mencionar el
  evento que sí se está esperando (macro, producto, etc.).
- **Comparación con playbook si aplica.** No forzar; solo si hay analogía real.

### Casos especiales

- **Activo nuevo (primer informe):** sin sección "Lo que cambió", incluir
  "Análisis inicial" más extenso.
- **Activo en watchlist (no en cartera):** sin "Tu posición". Foco en si se
  cumple el criterio de entrada.
- **Datos insuficientes:** declararlo arriba. No inventar. Sugerir esperar al
  próximo earnings o ampliar fuente de datos.

## Informe de cartera

### Estructura

```markdown
# Informe de cartera — {fecha}

## 🌡️ Termómetro general
{actitud general: "momento de no hacer nada" / "oportunidad concreta en X" / etc.}

## 📊 Estado vs límites
- Posiciones activas: {N} / {max}
- Sub-estrategia growth_moderado: {N} / {max}
- Reserva oportunidades: {disponible} / {target_range}
- Concentración sectorial: {top sectores con %}

## 🟢 Posiciones en buen estado
{lista compacta con P&L y estado de tesis}

## 🟡 Posiciones en seguimiento
{las que requieren atención: tesis en riesgo, cerca de niveles, etc.}

## 🔴 Candidatas a rotación
{las que deberían salir y por qué}

## 📅 Calendario próximas 4 semanas
{eventos relevantes: earnings, eventos macro, etc.}

## 🎯 Recomendaciones priorizadas
{lista numerada de acciones concretas con prioridad}
```

### Reglas de generación

- **Termómetro general primero.** Una frase que resuma la actitud del momento.
- **Priorizar acciones, no diagnóstico.** El usuario quiere saber qué hacer.
- **Detectar saturación.** Si hay >max_positions, candidata obligatoria.
- **Detectar oportunidades.** Si hay reserva de oportunidades sin desplegar y
  hay activos en watchlist en zona de entrada, indicarlo.

## Trigger sources

| Trigger                  | Genera informe       | Notifica                |
|--------------------------|----------------------|-------------------------|
| Cron semanal             | Sí, todos            | Resumen agregado        |
| Manual desde dashboard   | Sí, el solicitado    | Solo en dashboard       |
| Alerta de precio         | Sí, ad-hoc           | Push inmediato          |
| Earnings T-7             | No, recordatorio     | Push                    |
| Earnings T-1             | Sí, pre-earnings     | Push                    |
| Earnings T+1             | Sí, post-earnings    | Push                    |
| Noticia muy relevante    | Sí, ad-hoc           | Push                    |
| Cambio en fundamentales  | Sí, ad-hoc           | Push                    |

## Almacenamiento y entrega

- Todos los informes se guardan en la tabla `reports` (markdown + structured).
- Se sincronizan a Notion si está configurado.
- Se renderizan en el dashboard.
- El push a Telegram/email es un resumen de 3-5 líneas con link al informe completo.

## Calidad y evaluación

Para validar la calidad del output, se mantendrán ejemplos manuales de referencia
en [`examples/`](../examples/). Cada cambio de prompt se evalúa contra estos
ejemplos antes de desplegarse.
