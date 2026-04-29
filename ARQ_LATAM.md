# Arquitectura individual — Módulo Millas / Dispersión

## Objetivo
Automatizar la valoración mensual de millas, su clasificación financiera y la validación contra presupuesto.

## Arquitectura técnica

```mermaid
flowchart TD
    A[Archivo mensual de millas] --> B[Capa de ingesta]
    B --> C[Validador de estructura]
    C --> D{¿Archivo válido?}
    D -- No --> E[Error controlado: archivo inválido]
    D -- Sí --> F[Conversión a JSON]
    F --> G[Conversión a Parquet]
    G --> H[Staging de procesamiento]

    H --> I[Motor de limpieza]
    I --> J[Normalizar beneficiario, concepto, millas, USD, fechas]
    J --> K[Motor TRM]
    L[TRM manual/API] --> K
    K --> M[Calcular valores COP]
    M --> N[Calcular costo unitario y total]

    N --> O[Motor de clasificación]
    P[Reglas de clasificación] --> O
    O --> Q{Categoría identificada?}
    Q -- No --> R[Reporte categoría desconocida]
    Q -- Sí --> S[Clasificar consumo]

    S --> T[Consolidar por categoría]
    T --> U[Consolidar por periodo]
    V[Base presupuestal] --> W[Validador presupuestal]
    U --> W
    W --> X[Reporte desviaciones]
    W --> Y[Resumen financiero]
    R --> Z[Carpeta de reportes]
    X --> Z
    Y --> Z
```

## Componentes requeridos

| Componente | Responsabilidad |
|---|---|
| Ingesta Millas | Leer archivo mensual descargado |
| Conversor JSON/Parquet | Estandarizar procesamiento rápido |
| Motor de limpieza | Normalizar campos de millas, USD, beneficiario y concepto |
| Motor TRM | Obtener TRM manual o vía API |
| Motor financiero | Calcular valores en COP, costo unitario y total |
| Motor de clasificación | Clasificar persona natural, persona jurídica, bonos, tarjetas y business |
| Validador presupuestal | Comparar consumo mensual contra presupuesto |
| Generador de reportes | Emitir resumen financiero y desviaciones |

## Estructura sugerida del módulo

```text
millas_dispersion/
├── input/
├── staging/
│   ├── json/
│   └── parquet/
├── masters/
│   ├── reglas_clasificacion.xlsx
│   ├── trm.xlsx
│   └── presupuesto.xlsx
├── output/
│   ├── resumen_financiero/
│   └── reportes/
├── logs/
└── src/
    ├── ingest.py
    ├── transform.py
    ├── trm_engine.py
    ├── classifier.py
    ├── budget_validator.py
    └── main.py
```

## Salidas esperadas

- Resumen financiero mensual.
- Archivo de consumo clasificado.
- Reporte de categorías desconocidas.
- Reporte de desviaciones presupuestales.
- Archivo Parquet procesado.
- Bitácora de ejecución.
