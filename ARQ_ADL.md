# Arquitectura individual — Módulo ADL

## Objetivo
Automatizar la legalización mensual de anticipos ADL, desde la recepción del XLSX del proveedor hasta la generación de plantilla final, validación contable, detección de rubros nuevos y cruce presupuestal.

## Arquitectura técnica

```mermaid
flowchart TD
    A[Archivo proveedor ADL XLSX] --> B[Capa de ingesta]
    B --> C[Validador de estructura]
    C --> D{¿Columnas requeridas existen?}
    D -- No --> E[Error controlado: plantilla inválida]
    D -- Sí --> F[Conversor XLSX a JSON]
    F --> G[Conversor JSON a Parquet]
    G --> H[Data Lake local / staging]

    H --> I[Motor de limpieza]
    I --> J[Normalizar fechas, moneda, NIT, rubro, valores]
    J --> K[Filtro Banco Bogotá]
    K --> L[Separación Base / IVA / Total]

    L --> M[Motor de enriquecimiento contable]
    N[Maestro de parámetros contables] --> M
    M --> O{¿Rubro reconocido?}
    O -- No --> P[Registrar rubro nuevo]
    P --> Q[Actualizar tabla maestra pendiente de parametrización]
    Q --> R[Reprocesar registros afectados]
    R --> M
    O -- Sí --> S[Datos enriquecidos]

    S --> T[Generador Hoja Base]
    S --> U[Generador Hoja Distribución]
    T --> V[Plantilla Excel ADL final]
    U --> V

    S --> W[Consolidador por proyecto]
    W --> X[Resultado consolidado Parquet]
    Y[Base presupuestal] --> Z[Validador presupuestal]
    X --> Z
    Z --> AA[Reporte de desviaciones]
    Z --> AB[Reporte de excedidos/no presupuestados]

    V --> AC[Salida Excel legalizada]
    AA --> AD[Carpeta de reportes]
    AB --> AD
    P --> AD
```

## Componentes requeridos

| Componente | Responsabilidad |
|---|---|
| Ingesta ADL | Leer archivo mensual XLSX del proveedor |
| Validador de estructura | Confirmar columnas mínimas requeridas |
| Conversor de formatos | Generar JSON y Parquet para procesamiento rápido |
| Motor de limpieza | Normalizar datos financieros y campos clave |
| Motor de exclusión | Mantener únicamente registros Banco Bogotá |
| Motor financiero | Separar base, IVA y total |
| Maestro contable | Parametrizar rubro, cuenta, material, orden, centro de costo y proyecto |
| Motor de rubros nuevos | Detectar, registrar y reprocesar servicios nuevos |
| Generador Excel | Alimentar Hoja Base y Hoja Distribución |
| Validador presupuestal | Comparar legalización contra presupuesto |
| Bitácora | Guardar errores, cambios, rubros nuevos y trazabilidad |

## Estructura sugerida del módulo

```text
adl/
├── input/
├── staging/
│   ├── json/
│   └── parquet/
├── masters/
│   ├── maestro_contable.xlsx
│   └── presupuesto.xlsx
├── output/
│   ├── excel_final/
│   └── reportes/
├── logs/
└── src/
    ├── ingest.py
    ├── transform.py
    ├── rules.py
    ├── budget_validator.py
    ├── excel_writer.py
    └── main.py
```

## Salidas esperadas

- Archivo Excel legalizado.
- Reporte de rubros nuevos.
- Reporte de diferencias presupuestales.
- Archivo Parquet procesado.
- Bitácora de ejecución.
