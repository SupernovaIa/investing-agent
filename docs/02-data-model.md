# 02 — Modelo de datos

Este documento describe las entidades principales y sus relaciones. Es la
referencia para implementar las migraciones de base de datos y los modelos
SQLAlchemy/Pydantic.

## Principios

1. **Capa objetiva (assets, snapshots, news, events)** es compartida entre
   todos los usuarios. Un mismo Oracle es un solo registro en `assets`.
2. **Capa subjetiva (positions, theses, alerts)** es por usuario.
3. **Strategy** es un objeto JSONB validado por schema, no una tabla relacional
   rígida.
4. **Versionado:** las tesis e informes son inmutables; los cambios crean nuevas
   versiones con `is_current` apuntando a la última.

## Entidades

### `users`

| Campo                  | Tipo                  | Notas |
|------------------------|-----------------------|-------|
| `id`                   | UUID PK               | |
| `name`                 | string                | |
| `email`                | string unique         | |
| `language`             | string (BCP-47)       | ej. `es-ES` |
| `timezone`             | string (IANA)         | ej. `Europe/Madrid` |
| `strategy_id`          | UUID FK → strategies  | strategy activa |
| `notification_channels`| JSONB                 | telegram_id, email, etc. |
| `preferences`          | JSONB                 | tono, formato, etc. |
| `created_at`           | timestamptz           | |

### `strategies`

| Campo            | Tipo                  | Notas |
|------------------|-----------------------|-------|
| `id`             | UUID PK               | |
| `name`           | string                | ej. "Javi v1" |
| `owner_user_id`  | UUID FK → users null  | null = template público |
| `version`        | int                   | incremental |
| `config`         | JSONB                 | ver `03-strategy-spec.md` |
| `is_active`      | bool                  | |
| `created_at`     | timestamptz           | |
| `updated_at`     | timestamptz           | |

### `assets` (globales)

| Campo         | Tipo            | Notas |
|---------------|-----------------|-------|
| `id`          | UUID PK         | |
| `ticker`      | string unique   | ej. `ORCL`, `BTC-USD` |
| `name`        | string          | |
| `type`        | enum            | `stock`, `etf`, `crypto`, `commodity` |
| `sector`      | string null     | |
| `industry`    | string null     | |
| `country`     | string null     | ISO-3166 |
| `currency`    | string          | ISO-4217, ej. `USD` |
| `exchange`    | string null     | |
| `is_active`   | bool            | desactivar deslistados |
| `created_at`  | timestamptz     | |

### `asset_snapshots`

Histórico de datos objetivos. Particionado mental por fecha; en SQLite es solo
una tabla, en Postgres podría particionarse en el futuro.

| Campo                | Tipo            | Notas |
|----------------------|-----------------|-------|
| `id`                 | UUID PK         | |
| `asset_id`           | UUID FK         | |
| `date`               | date            | UNIQUE con `asset_id` |
| `price`              | numeric         | cierre |
| `ath`                | numeric null    | all-time high conocido |
| `ath_date`           | date null       | |
| `drawdown_from_ath`  | numeric null    | calculado, % |
| `low_52w`            | numeric null    | |
| `high_52w`           | numeric null    | |
| `volume`             | bigint null     | |
| `market_cap`         | numeric null    | |
| `pe_trailing`        | numeric null    | |
| `pe_forward`         | numeric null    | |
| `peg`                | numeric null    | |
| `eps_trailing`       | numeric null    | |
| `dividend_yield`     | numeric null    | |
| `analyst_consensus`  | string null     | strong_buy/buy/hold/sell/strong_sell |
| `avg_price_target`   | numeric null    | |
| `extra`              | JSONB           | métricas opcionales (ARR, RPO, NRR, etc.) |
| `source`             | string          | yfinance/fmp/alpha |
| `ingested_at`        | timestamptz     | |

### `asset_news`

| Campo            | Tipo            | Notas |
|------------------|-----------------|-------|
| `id`             | UUID PK         | |
| `asset_id`       | UUID FK         | |
| `published_at`   | timestamptz     | |
| `headline`       | string          | |
| `summary`        | text null       | |
| `url`            | string          | |
| `source`         | string          | |
| `sentiment`      | enum null       | bull/bear/neutral |
| `relevance_score`| numeric null    | 0..1 |
| `tags`           | string[] null   | ej. ['earnings', 'm&a'] |

### `events`

Catalizadores planeados o pasados.

| Campo         | Tipo            | Notas |
|---------------|-----------------|-------|
| `id`          | UUID PK         | |
| `asset_id`    | UUID FK null    | null si es evento macro |
| `type`        | enum            | earnings, dividend, split, m&a, ceo_change, macro |
| `date`        | timestamptz     | |
| `description` | text            | |
| `actuals`     | JSONB null      | resultado real cuando ocurra |
| `consensus`   | JSONB null      | expectativas previas |

### `positions`

| Campo               | Tipo            | Notas |
|---------------------|-----------------|-------|
| `id`                | UUID PK         | |
| `user_id`           | UUID FK         | |
| `asset_id`          | UUID FK         | UNIQUE con user_id+status_active |
| `status`            | enum            | active / watchlist / closed |
| `shares`            | numeric         | 0 si watchlist |
| `cost_basis_avg`    | numeric null    | |
| `total_invested`    | numeric null    | |
| `currency`          | string          | |
| `sub_strategy_id`   | string          | id dentro de la strategy |
| `opened_at`         | timestamptz null| |
| `closed_at`         | timestamptz null| |

### `theses`

Versionadas. Cuando el usuario edita su tesis, se crea una nueva versión y se
marca la anterior como `is_current = false`.

| Campo                  | Tipo            | Notas |
|------------------------|-----------------|-------|
| `id`                   | UUID PK         | |
| `position_id`          | UUID FK         | |
| `version`              | int             | |
| `bull_case`            | text            | |
| `bear_case`            | text            | |
| `breaking_conditions`  | text            | qué rompería la tesis |
| `reference_playbook`   | string null     | ej. "META 2022" |
| `conviction`           | int             | 1..5 |
| `entry_ladder`         | JSONB null      | niveles de adición planeados |
| `exit_plan`            | JSONB null      | niveles de salida |
| `target_price_optimistic` | numeric null | |
| `time_horizon_months`  | int null        | |
| `is_current`           | bool            | |
| `created_at`           | timestamptz     | |
| `notes`                | text null       | libre |

### `alerts`

| Campo         | Tipo            | Notas |
|---------------|-----------------|-------|
| `id`          | UUID PK         | |
| `user_id`     | UUID FK         | |
| `asset_id`    | UUID FK         | |
| `type`        | enum            | price_below, price_above, drawdown_above, earnings_in_days, news_relevant |
| `condition`   | JSONB           | parámetros del trigger |
| `status`      | enum            | active, triggered, dismissed, expired |
| `triggered_at`| timestamptz null| |
| `note`        | text null       | libre, ej. "3ª acción si toca" |
| `created_at`  | timestamptz     | |

### `reports`

| Campo                  | Tipo            | Notas |
|------------------------|-----------------|-------|
| `id`                   | UUID PK         | |
| `user_id`              | UUID FK         | |
| `asset_id`             | UUID FK null    | null si es informe de cartera |
| `type`                 | enum            | asset, portfolio |
| `trigger`              | enum            | scheduled_weekly, manual, alert_triggered, event_triggered |
| `prompt_version`       | string          | trazabilidad |
| `inputs_hash`          | string          | hash de los inputs para detectar duplicados |
| `content_markdown`     | text            | informe legible |
| `content_structured`   | JSONB           | secciones parseables |
| `recommendation_summary`| text           | one-liner accionable |
| `recommendation_type`  | enum null       | hold, add, trim, exit, watch |
| `generated_at`         | timestamptz     | |

### `report_deltas`

Para no repetir lo mismo informe tras informe. Almacena qué cambió respecto al
informe anterior.

| Campo            | Tipo            | Notas |
|------------------|-----------------|-------|
| `id`             | UUID PK         | |
| `report_id`      | UUID FK         | |
| `prev_report_id` | UUID FK null    | |
| `changes`        | JSONB           | secciones modificadas, métricas que se movieron |

## Relaciones (resumen)

```
users (1) ──── (1) strategies
users (1) ──── (N) positions ──── (1) assets
positions (1) ──── (N) theses [versionadas]
users (1) ──── (N) alerts ──── (1) assets
users (1) ──── (N) reports
assets (1) ──── (N) asset_snapshots
assets (1) ──── (N) asset_news
assets (1) ──── (N) events
```

## Índices recomendados (mínimos)

- `asset_snapshots(asset_id, date)` único
- `asset_news(asset_id, published_at)`
- `events(asset_id, date)`
- `positions(user_id, status)`
- `theses(position_id, is_current)`
- `alerts(user_id, status)`
- `reports(user_id, asset_id, generated_at)`
