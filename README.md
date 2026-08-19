# ☕ Cafetería ABC — Dashboard Ejecutivo de Rendimiento

Dashboard de Power BI para el análisis operativo y financiero de **Cafetería ABC**, una cadena con 5 sucursales entre Querétaro y CDMX. Construido en formato **PBIP** (Power BI Project), lo que permite versionarlo en Git como código en vez de manejar un `.pbix` binario.

![Cafetería ABC](Cafetería%20ABC.png)

## Qué muestra el dashboard

- **KPIs principales**: Ventas Netas, Ganancia Bruta, Margen %, Ticket Promedio y Descuentos, cada uno comparado contra el mes anterior.
- **Análisis 80/20 (Pareto)** de productos por ventas netas, con el porcentaje acumulado para identificar qué productos concentran la mayor parte de los ingresos.
- **Top productos más rentables**, ordenados por margen %.
- **Ventas por sucursal**, **distribución por canal de venta** (Mostrador / Llevar / Delivery) y **desglose por método de pago**.
- Filtros por periodo, ciudad, sucursal y categoría de producto.

El diseño visual (paleta "café y crema", tipografía Inter, tarjetas redondeadas) sigue el mockup en [`mockup_diseño/`](mockup_diseño), aplicado mediante un tema personalizado de Power BI.

## Estructura del repositorio

```
project_bi/
├── Datos/                          # Fuentes de datos (CSV)
│   ├── Ventas.csv                  # 5,000 transacciones (ene–may 2026)
│   ├── Productos.csv               # Catálogo de 15 productos
│   └── Sucursales.csv              # 5 sucursales (Querétaro y CDMX)
├── PBI/                            # Proyecto de Power BI (formato PBIP)
│   ├── proyecto.pbip               # Archivo raíz — ábrelo con Power BI Desktop
│   ├── proyecto.SemanticModel/     # Modelo de datos (tablas, relaciones, medidas DAX en TMDL)
│   ├── proyecto.Report/            # Definición del reporte y sus visuales
│   ├── tema_personalizado_abc.json # Tema visual (paleta y tipografía de marca)
│   └── plan_implementacion_dashboard.md  # Plan de construcción del dashboard
└── mockup_diseño/                  # Referencia de diseño (mockup, guía de estilo)
```

## Modelo de datos

Esquema en estrella con una tabla de hechos y tres dimensiones:

- **Ventas** (hechos) → relacionada con **Productos**, **Sucursales** y **Calendario**.
- **_Medidas**: tabla auxiliar con las métricas DAX (Ventas Netas, Ganancia Bruta, Margen %, Ticket Promedio, variaciones vs. mes anterior, ranking y % acumulado para el Pareto, participación por canal y método de pago).

## Cómo abrirlo

1. Instala [Power BI Desktop](https://www.microsoft.com/es-es/power-platform/products/power-bi/desktop) (gratis).
2. Clona este repositorio.
3. Abre `PBI/proyecto.pbip`.
4. Si Power BI no encuentra los CSV automáticamente, actualiza el parámetro `RutaDatos` (Inicio → Transformar datos → Editar parámetros) para que apunte a tu copia local de la carpeta `Datos/`.
5. Inicio → Actualizar, para cargar los datos.

## Stack

Power BI Desktop · Power Query (M) · DAX · formato PBIP/TMDL para control de versiones.

---

Datos de ejemplo con fines demostrativos — no representan cifras reales de un negocio.
