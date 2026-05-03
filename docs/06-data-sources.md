# 06 — Fuentes de datos

> 🚧 Documento inicial. Las fuentes concretas y sus precios se actualizarán
> tras hacer pruebas reales con cada una.

## Principio

Para cada métrica que necesitamos, identificamos:

1. **Fuente primaria** (la mejor disponible)
2. **Fallback** (alternativa si la primaria falla o no cubre el activo)
3. **Coste** (gratis / freemium / pago)
4. **Latencia aceptable** (real-time / EOD / semanal)

Las fuentes de datos son **adapters** intercambiables en el código. La capa de
ingesta no depende de yfinance específicamente; puede sustituirse por FMP sin
tocar el resto del sistema.

## Mapa de datos a fuentes (propuesta)

### Datos de mercado (precio, ATH, volumen)

| Métrica           | Primaria  | Fallback     | Coste        |
|-------------------|-----------|--------------|--------------|
| Precio EOD        | yfinance  | Alpha Vantage| Gratis       |
| Histórico ATH     | yfinance  | calculado    | Gratis       |
| 52w high/low      | yfinance  | -            | Gratis       |
| Volumen           | yfinance  | -            | Gratis       |

### Fundamentales

| Métrica           | Primaria  | Fallback           | Coste     |
|-------------------|-----------|--------------------|-----------|
| Market cap        | yfinance  | FMP                | Gratis/freemium |
| P/E trailing/fwd  | yfinance  | FMP                | Gratis/freemium |
| EPS               | yfinance  | FMP                | Gratis/freemium |
| Free cash flow    | FMP       | yfinance (limitado)| Freemium  |
| Márgenes          | FMP       | -                  | Freemium  |
| Deuda neta        | FMP       | -                  | Freemium  |
| ROE / ROIC        | FMP       | -                  | Freemium  |

### Métricas SaaS/tech específicas

ARR, RPO, NRR, etc. **No suelen estar en APIs estándar.** Se extraen de:

- Earnings releases (parsing manual o LLM-assisted)
- Investor relations sites
- Filings SEC (10-Q, 10-K)

Decisión MVP: **omitir.** Se añadirán como datos opcionales declarados
manualmente por el usuario o extraídos por el LLM al leer earnings.

### Calendario de earnings

| Fuente                | Coste     | Notas |
|-----------------------|-----------|-------|
| yfinance (calendar)   | Gratis    | Limitado pero suficiente |
| FMP earnings calendar | Freemium  | Más completo |
| Investing.com scrape  | Gratis    | Riesgoso, depende del scrape |

### Sentimiento de analistas

| Métrica                | Primaria | Fallback | Coste |
|------------------------|----------|----------|-------|
| Consensus rating       | yfinance | FMP      | Gratis|
| Avg price target       | yfinance | FMP      | Gratis|
| Revisiones recientes   | FMP      | -        | Freemium |

### Insider trading

| Métrica          | Primaria             | Coste |
|------------------|----------------------|-------|
| Insider buys/sells | FMP / SEC EDGAR    | Gratis (con esfuerzo) |

### Noticias

| Fuente              | Coste      | Notas |
|---------------------|------------|-------|
| Yahoo Finance RSS   | Gratis     | Suficiente para MVP |
| Google News RSS     | Gratis     | Buena cobertura |
| FMP news endpoint   | Freemium   | Mejor calidad |
| Newsapi.org         | Freemium   | Más amplio |

### Datos macro

| Métrica                | Fuente   | Coste |
|------------------------|----------|-------|
| S&P 500 nivel actual   | yfinance | Gratis|
| Tipos de interés FED   | FRED API | Gratis|
| Inflación CPI          | FRED API | Gratis|
| EUR/USD                | yfinance | Gratis|

### Datos de cripto

| Métrica           | Fuente              | Coste |
|-------------------|---------------------|-------|
| Precio crypto     | yfinance / CoinGecko| Gratis|
| Histórico         | CoinGecko           | Gratis (con rate limits) |

## Estrategia MVP

**Fase 1 — Solo gratis:**
- `yfinance` para casi todo (precio, fundamentales básicos, earnings, consensus)
- `FRED API` para macro
- RSS feeds para noticias

Esto cubre el ~80% de lo que el agente necesita y permite validar el sistema
sin costes recurrentes.

**Fase 2 — Freemium cuando sea necesario:**
- FMP (~$15-30/mes plan básico) para métricas avanzadas y fiabilidad
- Newsapi para mejor cobertura de noticias

**Fase 3 — Si escala:**
- Datos premium específicos según necesidad

## Rate limits y caching

- Cachear precios EOD por día (no volver a pedir lo ya tirado)
- Cachear fundamentales por trimestre (refrescar tras earnings)
- Cachear noticias por 1-6h
- Implementar backoff exponencial en todos los adapters

## Confiabilidad

Para cada fuente:
- Test de salud diario (¿responde? ¿tiempos razonables?)
- Comparación cruzada de precio entre fuentes (alerta si discrepancia >1%)
- Marca el `source` en `asset_snapshots` para auditoría
