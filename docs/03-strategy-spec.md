# 03 — Especificación de Strategy

Una **Strategy** es la representación estructurada del framework de inversión
de un usuario. Es lo que permite que el agente sea un motor genérico aplicable
a perfiles muy distintos.

## Estructura general

Una strategy es un objeto JSON/YAML con los siguientes bloques:

```yaml
metadata:           # Información identificativa
sub_strategies:     # Lista de sub-estrategias (buckets)
global_rules:       # Reglas que aplican a la cartera global
playbooks:          # Casos históricos de referencia
crash_plan:         # Plan ante correcciones de mercado
crypto_plan:        # Plan específico para criptomonedas (opcional)
analysis_style:     # Cómo debe razonar el LLM
```

Ver [`schemas/strategy.schema.json`](../schemas/strategy.schema.json) para la
definición formal y [`strategies/javi-v1.yaml`](../strategies/javi-v1.yaml)
para el ejemplo de referencia.

## Bloques

### `metadata`

```yaml
metadata:
  name: "Javi v1"
  version: 1
  language: "es-ES"
  description: |
    Estrategia de un inversor con núcleo pasivo en Indexa Capital +
    cartera contrarian/growth en acciones individuales + DCA en cripto.
```

### `sub_strategies`

Una strategy puede contener varias sub-estrategias simultáneamente. Cada activo
en cartera se asigna a una de ellas.

Cada sub-estrategia tiene:

```yaml
- id: "growth_moderado"            # identificador estable
  name: "Estrategia 2 - Growth moderado"
  type: "discretionary"            # passive_dca | discretionary | dca_with_triggers
  enabled: true

  entry_criteria:                  # qué debe cumplir un activo para entrar
    drawdown_from_ath_min: 20
    drawdown_from_ath_max: 30
    fundamentals: "intact_or_improving"
    quality_tier: "blue_chip"
    catalyst_required: true

  position_sizing:
    typical_eur: [400, 600]
    max_eur: 1000
    scales_with: "discount_depth"  # cómo escalar la posición

  max_positions: 3                 # tope dentro del bucket

  exit_criteria:
    - condition: "near_ath"
      action: "sell_50_to_100_pct"
    - condition: "thesis_broken"
      action: "cut_losses"
    - condition: "extraordinary_gain"
      threshold_pct: 60
      action: "sell_partial_or_full"

  notes: |
    Notas libres del usuario sobre esta sub-estrategia.
```

Tipos de sub-estrategia soportados:

- **`passive_dca`** — aportación periódica fija a un fondo/ETF, sin selección
  activa de activos. Ej: Indexa Capital.
- **`discretionary`** — selección activa de activos individuales según criterios.
  Ej: contrarian / growth en acciones.
- **`dca_with_triggers`** — DCA base + adiciones extra cuando se cumplen
  triggers. Ej: BTC 100€/mes + 1000€ extra si baja de 40k.
- **`lottery`** — posiciones pequeñas pasivas sin gestión activa. Ej: altcoins
  especulativas.

### `global_rules`

Reglas que aplican a toda la cartera, no a una sub-estrategia concreta.

```yaml
global_rules:
  max_active_positions: 10
  max_per_sub_strategy:
    growth_moderado: 3
  opportunity_cash_reserve_eur: [5000, 6000]
  sector_concentration_max_pct: 40   # opcional
```

### `playbooks`

Casos históricos que el usuario usa como analogía. El LLM puede invocarlos en
sus análisis para anclar recomendaciones.

```yaml
playbooks:
  success:
    - ticker: "META"
      year: 2022
      lesson: "Pánico metaverso, fundamentales intactos → 4x"
    - ticker: "TSLA"
      lesson: "Corrección fuerte con tesis intacta → 2x"

  cautionary:
    - ticker: "NKE"
      lesson: "Marca fuerte ≠ inversión sólida. Promediar a la baja sin tesis clara."
    - ticker: "ASTON_MARTIN"
      lesson: "Value trap, no contrarian. Deuda insostenible y negocio sangrando."
```

### `crash_plan`

Plan escalonado para aprovechar correcciones de mercado.

```yaml
crash_plan:
  benchmark: "SP500"
  ladder:
    - drawdown_pct: 20
      action: "deploy_eur"
      amount: 1500
      target: "indexa_core"
    - drawdown_pct: 30
      action: "deploy_eur"
      amount: 2000
      target: "indexa_core"
    - drawdown_pct: 40
      action: "deploy_remaining"
      target: "indexa_core"
```

### `crypto_plan` (opcional)

```yaml
crypto_plan:
  dca_monthly:
    BTC: 100
    ETH: 50
    SOL: 50
  cashback_to: "BTC"
  hodl_only: ["BTC"]
  crash_triggers:
    - asset: "BTC"
      condition: "price_below"
      threshold: 40000
      currency: "USD"
      deploy_eur: 1000
    - asset: "ETH"
      condition: "price_below"
      threshold: 1500
      currency: "USD"
      deploy_eur: 500
  passive_lottery: ["LINK", "NEAR"]
```

### `analysis_style`

Define cómo debe razonar y comunicar el LLM. Es lo que hace que los análisis
mantengan el estilo del usuario, no genéricos.

```yaml
analysis_style:
  tone: "directo, conciso, sin paja"
  formality: "tutear"
  always_include:
    - "% drawdown desde ATH antes de opinar"
    - "Clasificación explícita en sub-estrategia"
    - "Bull case y bear case honestos"
    - "Recomendación cuantificada (cantidades, niveles)"
    - "Próximo catalizador con fecha"
  reference_playbooks: true
  red_flags_to_call_out:
    - "Activos sin tesis bear identificada"
    - "Acercarse a max_positions"
    - "Concentración sectorial alta"
```

## Validación

Toda strategy se valida contra `schemas/strategy.schema.json` antes de ser
aceptada por el sistema. Una strategy inválida no puede activarse.

## Versionado

Cada vez que el usuario edita su strategy, se incrementa `version` y se guarda
la nueva como entrada distinta. Los informes históricos referencian la versión
de strategy con la que se generaron, para trazabilidad.

## Sharing y templates

Una strategy con `owner_user_id = null` es una **plantilla pública** que otros
usuarios pueden clonar como punto de partida. Ejemplos previstos:

- `dividend-investor-template.yaml`
- `value-graham-style.yaml`
- `crypto-maxi.yaml`
- `bogleheads-passive.yaml`

Esto sale del scope del MVP pero la arquitectura lo soporta desde el día uno.
