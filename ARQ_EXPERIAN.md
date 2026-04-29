# Arquitectura individual — Módulo Experian

## Objetivo
Automatizar la conciliación de facturas y consumos Experian para los subprocesos Valor Ingreso, Línea y Batch.

## Arquitectura técnica

```mermaid
flowchart TD
    A[Inicio módulo Experian] --> B{Tipo de insumo}

    B --> VI0[Valor ingreso]
    B --> LN0[Línea]
    B --> BT0[Batch]

    %% Valor ingreso
    subgraph VI[Subflujo Valor Ingreso]
        VI0 --> VI1[Ingestar factura/base]
        VI1 --> VI2[Validar estructura]
        VI2 --> VI3[Convertir a JSON/Parquet]
        VI3 --> VI4[Extraer cantidades facturadas]
        VI5[Grafana export/API] --> VI6[Normalizar consumo operacional]
        VI4 --> VI7[Motor conciliador]
        VI6 --> VI7
        VI7 --> VI8{¿Coincide?}
        VI8 -- Sí --> VI9[Conciliación aprobada]
        VI8 -- No --> VI10[Reporte de diferencias]
    end

    %% Línea
    subgraph LN[Subflujo Línea]
        LN0 --> LN1[Ingestar salida de script]
        LN1 --> LN2[Normalizar columnas producto/oficina/cantidad]
        LN2 --> LN3[Validar oficinas]
        LN3 --> LN4{¿Existen consumos desconocidos?}
        LN4 -- Sí --> LN5[Aplicar motor de prorrateo]
        LN5 --> LN6[Asignar oficina definitiva]
        LN4 -- No --> LN6
        LN6 --> LN7[Consolidar producto/oficina/cantidad]
        LN7 --> LN8[Generar archivo final]
    end

    %% Batch
    subgraph BT[Subflujo Batch]
        BT0 --> BT1[Ingestar base batch]
        BT1 --> BT2[Convertir a JSON/Parquet]
        BT2 --> BT3[Extraer IDs / identificaciones]
        BT4[Base productividad / portal] --> BT5[Normalizar maestro productividad]
        BT3 --> BT6[Cruce por ID]
        BT5 --> BT6
        BT6 --> BT7{¿ID encontrado?}
        BT7 -- No --> BT8[Reporte IDs no encontrados]
        BT7 -- Sí --> BT9[Asignar oficina]
        BT9 --> BT10[Registrar visto bueno]
        BT10 --> BT11[Consolidar oficina/producto/cantidad]
    end

    VI9 --> OUT[Salida Experian]
    VI10 --> OUT
    LN8 --> OUT
    BT8 --> OUT
    BT11 --> OUT
```

## Componentes requeridos

| Componente | Responsabilidad |
|---|---|
| Ingesta Experian | Leer facturas y bases de los tres subprocesos |
| Conversor JSON/Parquet | Estandarizar procesamiento rápido |
| Conector Grafana | Recibir exporte o consumir API de Grafana |
| Motor conciliador | Comparar facturado vs consumo operacional |
| Normalizador de script Línea | Estandarizar salida de terceros |
| Motor de prorrateo | Distribuir consumos desconocidos por porcentaje |
| Conector productividad | Cruzar IDs contra información de oficina/usuario |
| Motor de consolidación | Agrupar por producto, oficina y cantidad |
| Reportador | Emitir diferencias, IDs no encontrados y consolidados |

## Estructura sugerida del módulo

```text
experian/
├── input/
│   ├── valor_ingreso/
│   ├── linea/
│   └── batch/
├── staging/
│   ├── json/
│   └── parquet/
├── masters/
│   ├── productividad.xlsx
│   ├── reglas_prorrateo.xlsx
│   └── oficinas.xlsx
├── output/
│   ├── conciliaciones/
│   └── reportes/
├── logs/
└── src/
    ├── valor_ingreso.py
    ├── linea.py
    ├── batch.py
    ├── conciliator.py
    ├── prorrateo.py
    └── main.py
```

## Salidas esperadas

- Conciliación Valor Ingreso.
- Consolidado Línea por producto/oficina/cantidad.
- Consolidado Batch por oficina/producto/cantidad.
- Reporte de consumos desconocidos.
- Reporte de IDs no encontrados.
- Bitácora de ejecución.
