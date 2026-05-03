# Strategies

Las estrategias de inversión viven aquí como YAML versionado.

## Estructura

```
strategies/
├── javi-v1.yaml              # Estrategia del primer usuario (referencia)
├── templates/                # Plantillas reutilizables (futuro)
│   ├── README.md
│   ├── dividend-investor.yaml
│   ├── bogleheads-passive.yaml
│   ├── value-graham.yaml
│   └── crypto-maxi.yaml
└── README.md
```

## Cómo crear una strategy nueva

1. Copia `javi-v1.yaml` como punto de partida.
2. Edita los campos según tu framework personal.
3. Valida contra `schemas/strategy.schema.json`:
   ```bash
   python -m scripts.validate_strategy strategies/mi-strategy.yaml
   ```
4. Carga en la base de datos:
   ```bash
   python -m scripts.load_strategy strategies/mi-strategy.yaml --user <user_id>
   ```

## Versionado

- Los cambios significativos a una strategy incrementan el campo `metadata.version`.
- En la base de datos, cada versión se persiste como entrada distinta.
- Los informes históricos referencian la versión con la que se generaron.

## Plantillas (futuro)

Las plantillas en `templates/` son strategies con `owner_user_id = null` que
otros usuarios pueden clonar como punto de partida. Cubrirán perfiles típicos:

- **dividend-investor**: foco en yield, DGI, payout ratio sano.
- **bogleheads-passive**: solo ETFs indexados, DCA mensual.
- **value-graham**: criterios cuantitativos clásicos (P/E < X, margen seguridad).
- **crypto-maxi**: BTC + altcoins, allocation por capas.

Estas plantillas se irán añadiendo conforme aparezcan usuarios que las pidan.

## Especificación

Ver [`docs/03-strategy-spec.md`](../docs/03-strategy-spec.md) para la
descripción completa de los bloques de una strategy.
