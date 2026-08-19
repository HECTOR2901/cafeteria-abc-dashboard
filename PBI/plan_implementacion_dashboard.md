# Plan de implementación — Dashboard "Cafetería ABC" en Power BI

Objetivo: replicar en Power BI Desktop, con la mayor fidelidad posible, el mockup `screen.png` / `DESIGN.md` de la carpeta `mockup_diseño`, usando los datos ya conectados de `Datos/Ventas.csv`, `Productos.csv` y `Sucursales.csv`, y el tema `tema_personalizado_abc.json` ya generado.

Estado actual del proyecto (punto de partida): modelo semántico con las 3 tablas conectadas y relacionadas (Ventas→Productos, Ventas→Sucursales), reporte con una página vacía de 1280×720, tema personalizado listo para aplicar.

Alcance y una decisión que debes tomar antes de empezar: el mockup muestra pestañas de navegación "Dashboard / Inventory / Sales / Staff", pero solo tenemos datos de ventas — no hay tablas de inventario ni de personal. Recomiendo construir solo la pestaña "Dashboard" como página funcional y dejar las otras tres como texto decorativo no interactivo (o quitarlas). Si más adelante consigues datos de inventario o staff, se agregan como páginas nuevas del mismo reporte.

## Fase 0 — Preparación

1. Confirmar que el tema `tema_personalizado_abc.json` está aplicado: Ver → Temas → Explorar temas → Examinar temas.
2. Si quieres tipografía 100% fiel, instalar la fuente Inter (Google Fonts) en Windows; si no, Power BI usará Segoe UI como respaldo, que es visualmente muy similar.
3. En Archivo → Opciones y configuración → Opciones → Datos actuales, desactivar "Detección automática de tablas de fechas" antes de crear la tabla Calendario manual (evita que Power BI genere jerarquías de fecha ocultas duplicadas).
4. Redimensionar la página: Formato de la página → Tamaño de página → Personalizado → **1280 × 1100** px (el mockup es más alto que ancho porque es un dashboard con scroll vertical, no una diapositiva 16:9).

## Fase 1 — Completar el modelo de datos

### 1.1 Tabla Calendario

El mockup filtra por "Periodo: Ene 2026 – May 2026" y muestra comparativas "vs mes anterior", así que se necesita una tabla de fechas contigua para que la inteligencia de tiempo de DAX funcione bien.

Crear una tabla calculada (Modelado → Nueva tabla):

```
Calendario =
CALENDAR(DATE(2026,1,1), DATE(2026,5,31))
```

Agregar columnas calculadas: `Año`, `Mes` (número), `NombreMes`, `AñoMes` (texto "2026-01" para ordenar), y marcar la columna `NombreMes` con "Ordenar por columna" = `Mes`.

Crear la relación **Calendario[Date] → Ventas[Fecha]** (uno a varios) y marcar la tabla como tabla de fechas: clic derecho en Calendario → "Marcar como tabla de fechas" → columna `Date`.

### 1.2 Medidas DAX — Núcleo de negocio

```
Ventas Brutas = SUMX(Ventas, Ventas[Cantidad] * Ventas[PrecioUnitario])

Descuentos = SUMX(Ventas, Ventas[Cantidad] * Ventas[PrecioUnitario] * Ventas[Descuento])

Ventas Netas = [Ventas Brutas] - [Descuentos]

Costo Total = SUMX(Ventas, Ventas[Cantidad] * RELATED(Productos[Costo]))

Ganancia Bruta = [Ventas Netas] - [Costo Total]

Margen % = DIVIDE([Ganancia Bruta], [Ventas Netas])

Unidades Vendidas = SUM(Ventas[Cantidad])

Numero Transacciones = DISTINCTCOUNT(Ventas[VentaID])

Ticket Promedio = DIVIDE([Ventas Netas], [Numero Transacciones])
```

### 1.3 Medidas DAX — Comparativas "vs mes anterior" (para las 5 tarjetas KPI)

```
Ventas Netas MA = CALCULATE([Ventas Netas], DATEADD(Calendario[Date], -1, MONTH))
Δ Ventas Netas % = DIVIDE([Ventas Netas] - [Ventas Netas MA], [Ventas Netas MA])

Ganancia Bruta MA = CALCULATE([Ganancia Bruta], DATEADD(Calendario[Date], -1, MONTH))
Δ Ganancia Bruta % = DIVIDE([Ganancia Bruta] - [Ganancia Bruta MA], [Ganancia Bruta MA])

Margen % MA = CALCULATE([Margen %], DATEADD(Calendario[Date], -1, MONTH))
Δ Margen pp = [Margen %] - [Margen % MA]

Ticket Promedio MA = CALCULATE([Ticket Promedio], DATEADD(Calendario[Date], -1, MONTH))
Δ Ticket Promedio % = DIVIDE([Ticket Promedio] - [Ticket Promedio MA], [Ticket Promedio MA])

Descuentos MA = CALCULATE([Descuentos], DATEADD(Calendario[Date], -1, MONTH))
Δ Descuentos % = DIVIDE([Descuentos] - [Descuentos MA], [Descuentos MA])
```

Nota: en el mockup, la tarjeta "Margen" muestra "-0.5%" con flecha roja — eso es una diferencia de **puntos porcentuales**, no un cambio relativo (por eso `Δ Margen pp` resta directamente, sin `DIVIDE`). Las demás tarjetas sí son variación relativa.

### 1.4 Medidas DAX — Gráfico Pareto 80/20 (Análisis de productos)

```
Ranking Producto = RANKX(ALL(Productos[Producto]), [Ventas Netas], , DESC, Dense)

Ventas Acumuladas =
VAR RankActual = [Ranking Producto]
RETURN
    CALCULATE(
        [Ventas Netas],
        FILTER(ALL(Productos[Producto]), [Ranking Producto] <= RankActual)
    )

% Acumulado = DIVIDE([Ventas Acumuladas], CALCULATE([Ventas Netas], ALL(Productos)))
```

### 1.5 Medidas DAX — Métodos de pago y canal (para el 100% stacked bar y el donut)

```
% del Total (Metodo Pago) =
DIVIDE([Ventas Netas], CALCULATE([Ventas Netas], ALL(Ventas[MetodoPago])))

% del Total (Canal) =
DIVIDE([Ventas Netas], CALCULATE([Ventas Netas], ALL(Ventas[CanalVenta])))
```

Organizar todas estas medidas en una tabla auxiliar "_Medidas" (Modelado → Nueva tabla vacía `_Medidas = {}` y luego mover las medidas ahí) para mantener el panel de Datos limpio, tal como recomienda buena práctica de modelado.

## Fase 2 — Estructura de página y filtros globales

Todas las coordenadas de abajo son valores exactos que puedes escribir directamente en el panel Formato → General → Propiedades de cada visual (X, Y, Ancho, Alto), extraídos y escalados 0.8× desde `screen.png` (1600×1280 → lienzo 1280×1024).

| Zona | X | Y | Ancho | Alto |
|---|---|---|---|---|
| Encabezado (rectángulo + logo + título "Cafetería ABC") | 0 | 0 | 1280 | 56 |
| Título "Dashboard Ejecutivo de Rendimiento" + subtítulo | 16 | 68 | 480 | 92 |
| Slicer "Periodo" | 496 | 68 | 260 | 44 |
| Slicer "Ciudad" | 776 | 68 | 176 | 44 |
| Slicer "Sucursal" | 968 | 68 | 176 | 44 |
| Slicer "Categoría" | 496 | 116 | 208 | 44 |

Construir el encabezado con: un rectángulo de fondo color `#FFF8F5` (0,0,1280,56), un ícono o imagen del logo (48×48 en x=16,y=4), texto "Cafetería ABC" en `largeTitle`/negrita.

Los 4 slicers deben ser de tipo lista desplegable ("Dropdown") con el estilo "Chip"/pastilla; el de "Periodo" puede ser un slicer de rango de fechas ("Entre") apuntando a `Calendario[Date]`. Aplicar Sincronización de filtros (Ver → Sincronizar segmentación) si luego agregas más páginas.

## Fase 3 — Fila de KPIs (5 tarjetas)

| # | Título | Medida valor | Medida delta | X | Ancho |
|---|---|---|---|---|---|
| 1 | VENTAS NETAS | `[Ventas Netas]` | `[Δ Ventas Netas %]` | 16 | 238 |
| 2 | GANANCIA BRUTA | `[Ganancia Bruta]` | `[Δ Ganancia Bruta %]` | 270 | 238 |
| 3 | MARGEN | `[Margen %]` | `[Δ Margen pp]` | 524 | 238 |
| 4 | TICKET PROMEDIO | `[Ticket Promedio]` | `[Δ Ticket Promedio %]` | 778 | 238 |
| 5 | DESCUENTOS | `[Descuentos]` | `[Δ Descuentos %]` | 1032 | 232 |

Todas en Y=168, Alto=72.

Usar el visual **"Tarjeta" (Card, versión nueva con formato condicional de referencia)** — no la tarjeta clásica — porque soporta de forma nativa una "etiqueta de referencia" con flecha, color y texto de comparación (exactamente el patrón "+5.2% vs mes anterior" del mockup). Configuración por tarjeta: Datos → medida de valor; Etiqueta de referencia → medida delta, con reglas de color (verde si ≥0, rojo si <0, gris/"Sin cambio" si =0) y formato de flecha activado. Formato de número: `[Ventas Netas]`, `[Ganancia Bruta]`, `[Ticket Promedio]`, `[Descuentos]` como moneda `$#,##0`; `[Margen %]` como porcentaje `0.0%`.

## Fase 4 — Fila 2: Pareto 80/20 y Top 5 rentables

| Visual | Tipo | X | Y | Ancho | Alto |
|---|---|---|---|---|---|
| Análisis 80/20 de Productos | Gráfico de líneas y columnas agrupadas | 16 | 256 | 732 | 320 |
| Top 5 Rentables | Tabla | 764 | 256 | 500 | 320 |

**Pareto**: Eje = `Productos[Producto]`, Columnas = `[Ventas Netas]`, Línea (eje secundario 0–100%) = `[% Acumulado]`. Filtro de nivel de visual: Top N = 5 por `[Ventas Netas]`. Ordenar el eje por `[Ventas Netas]` descendente. Color de columna = dataColors[0] (espresso), color de línea = rojo/error (`#BA1A1A`, como en el mockup) con marcadores circulares activados.

**Top 5 Rentables**: tabla con columnas Producto, Categoría, `[Unidades Vendidas]`, `[Margen %]`. Filtro de nivel de visual: Top N = 5 por `[Margen %]`. Formato de tabla: sin líneas verticales, separador horizontal `#F0EEEC`, encabezados en negrita `#1E1B18`.

## Fase 5 — Fila 3: Sucursal, Canal y Métodos de pago

| Visual | Tipo | X | Y | Ancho | Alto |
|---|---|---|---|---|---|
| Ventas por Sucursal | Barras agrupadas horizontales | 16 | 592 | 340 | 434 |
| Distribución por Canal | Gráfico de anillos (donut) | 368 | 592 | 340 | 434 |
| Métodos de Pago | Barras 100% apiladas horizontales | 720 | 592 | 544 | 434 |

**Ventas por Sucursal**: Eje = `Sucursales[Sucursal]`, Valores = `[Ventas Netas]`, ordenado descendente, etiquetas de datos activadas fuera de la barra, color único = dataColors[1] (crema).

**Distribución por Canal**: Leyenda = `Ventas[CanalVenta]`, Valores = `[Ventas Netas]`, etiquetas de datos en % activadas, leyenda arriba, agujero del donut ~60%.

**Métodos de Pago**: Eje = ninguno (una sola categoría; puedes usar una columna constante o simplemente dejar el eje vacío para una sola barra), Leyenda = `Ventas[MetodoPago]`, Valores = `[Ventas Netas]`, 100% apilado, etiquetas de datos en % dentro del centro de cada segmento, leyenda abajo.

## Fase 6 — Aplicar tema y afinar detalles visuales

1. Reconfirmar que el tema `tema_personalizado_abc.json` sigue aplicado (los colores de datos, tarjetas y tabla ya están definidos ahí).
2. Revisar manualmente los radios de esquina (16px en tarjetas grandes, según `DESIGN.md`) y la sombra "Soft Bloom" — el tema ya los aplica globalmente, pero verifica visual por visual que no se hayan sobreescrito.
3. Iconos del encabezado (campana, engrane) y del logo: Power BI no trae un set de iconos idéntico a Lucide/Feather del mockup. Opciones: (a) insertar los iconos como imágenes PNG/SVG con fondo transparente, o (b) usar caracteres Unicode/emoji como aproximación rápida. Recomiendo (a) para fidelidad real.
4. Botón "Export Report": es decorativo salvo que publiques a Power BI Service, donde sí existe exportación nativa a PDF/PowerPoint (Archivo → Exportar, en el servicio). En Desktop, simúlalo como una forma rectangular con texto, sin acción real, o con un marcador (bookmark) si quieres que alterne una vista de exportación.

## Fase 7 — Interactividad y pulido final

1. Editar interacciones (Formato → Editar interacciones) para que los filtros/slicers y los clics en el donut o el gráfico de sucursales resalten (no filtren) el resto de los visuales, igual que se espera de un dashboard ejecutivo.
2. Configurar tooltips personalizados si se desea (página oculta con tarjeta de detalle) para el Pareto y el donut.
3. Verificar accesibilidad: contraste de texto sobre fondo crema (`#51443E` sobre `#FFFFFF`/`#FFF8F5` cumple WCAG AA), tamaños de fuente mínimos 9pt para etiquetas.
4. Guardar una copia de seguridad del `.pbip` antes de cada cambio grande, dado que el modelo se sigue editando también a mano vía TMDL.

## Fase 8 — Verificación de fidelidad

1. Poner el mockup `screen.png` en un monitor/ventana al lado del reporte en Power BI Desktop (vista Lectura, con la barra de filtros de Power BI oculta) y comparar zona por zona: colores exactos, proporciones de las tarjetas, alineación de textos.
2. Validar que las cifras calculadas (Ventas Netas, Margen, etc.) sean coherentes con los datos reales de `Ventas.csv` — las cifras del mockup son solo de referencia visual, no tienen que coincidir exactamente porque son datos de ejemplo del diseñador.
3. Probar los filtros (Periodo, Ciudad, Sucursal, Categoría) para confirmar que todos los visuales reaccionan correctamente y que las medidas de comparación "vs mes anterior" no rompen en los bordes del rango de fechas (enero no tiene mes anterior con datos: debe mostrarse en blanco o "N/D", no un error).

---

Orden recomendado de ejecución: Fase 0 → 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8. Las fases 3, 4 y 5 (los visuales) se pueden trabajar en paralelo una vez completado el modelo de datos (Fase 1), pero conviene aplicar el tema (Fase 6, punto 1) antes de empezar a insertar visuales, así cada uno nace ya con el estilo correcto en vez de tener que reformatear todo al final.
