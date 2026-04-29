# Arquitectura core — Aplicación de automatización financiera

## Objetivo
Definir la arquitectura transversal reutilizable para los tres módulos: ADL, Experian y Millas/Dispersión.

```mermaid
flowchart LR
    U[Usuario funcional] --> UI[Interfaz / CLI / Panel de ejecución]
    UI --> ORQ[Orquestador]

    ORQ --> ADL[Módulo ADL]
    ORQ --> EXP[Módulo Experian]
    ORQ --> MIL[Módulo Millas]

    ADL --> CORE
    EXP --> CORE
    MIL --> CORE

    subgraph CORE[Core reutilizable]
        C1[Ingesta de archivos]
        C2[Validación de estructura]
        C3[Conversión JSON/Parquet]
        C4[Limpieza y normalización]
        C5[Motor de reglas]
        C6[Motor de cruces]
        C7[Motor presupuestal]
        C8[Generador Excel/reportes]
        C9[Logs y auditoría]
    end

    CORE --> OUT[Repositorio de salidas]
    OUT --> R1[Excel final]
    OUT --> R2[Reportes de incidencias]
    OUT --> R3[Archivos Parquet]
    OUT --> R4[Bitácoras]
```

## Estructura global sugerida del proyecto

```text
automatizacion_financiera/
├── app/
│   ├── orchestrator.py
│   ├── config.py
│   └── main.py
├── core/
│   ├── ingestion.py
│   ├── validators.py
│   ├── converters.py
│   ├── cleaners.py
│   ├── rules_engine.py
│   ├── joins.py
│   ├── budget.py
│   ├── reports.py
│   └── logger.py
├── modules/
│   ├── adl/
│   ├── experian/
│   └── millas_dispersion/
├── data/
│   ├── input/
│   ├── staging/
│   ├── masters/
│   └── output/
├── logs/
├── tests/
├── requirements.txt
└── README.md
```

## Principios de diseño

- Cada módulo debe usar el mismo core.
- Cada flujo debe producir JSON y Parquet antes de procesar grandes volúmenes.
- Las reglas de negocio deben estar parametrizadas en archivos maestros.
- Todo error debe quedar en bitácora.
- Ningún proceso debe depender de copiar y pegar manualmente.
- Los reportes deben ser reproducibles y auditables.
