# Plan de implementación — Dashboard "Cafetería ABC" en Power BI

**Estado: implementado.** Este documento empezó como plan de construcción (replicar el mockup de `mockup_diseño/` en Power BI Desktop) y se actualizó con lo que finalmente se construyó. Los resultados y hallazgos están en el [README](README.md).

## Qué cambió respecto al plan inicial

| Tema | Plan inicial | Resultado final |
|---|---|---|
| KPIs | 5 tarjetas con comparativa "vs mes anterior" | **6 tarjetas** (se agregó **Ventas por Día**), solo con valores |
| Comparativa vs mes anterior | Flecha y % en cada tarjeta | **No se muestra.** Febrero tiene 28 días y mayo llega solo hasta el día 25, así que comparar totales mensuales engaña. Las medidas siguen en el modelo (`_Medidas`) |
| Pareto 80/20 | Top 5 productos | **Los 15 productos**, para ver cuántos hacen falta para llegar al 80% |
| Tabla de productos | "Top 5 Rentables" | Tabla con los 15 productos ordenada por margen % (con scroll) |
| Pestañas del mockup (Inventory / Sales / Staff) | Decorativas o eliminadas | **Eliminadas**: solo hay datos de ventas |
| Filtros | Periodo, Ciudad, Sucursal, Categoría | Los mismos 4: **fecha** (rango "Entre"), ciudad, categoría y sucursal |
| Sucursales | Barras color crema | Barras color espresso, con etiquetas de datos |
| Línea del Pareto | Roja (`#BA1A1A`) | Color crema del tema |
| Medidas de visuales | Coordenadas exactas escaladas desde el mockup | Se ajustó el layout a mano; las coordenadas del plan inicial ya no aplican |

## Fase 0 — Preparación

1. Aplicar el tema `tema_personalizado_abc.json`: Vista → Temas → Examinar temas.
   - Si Power BI muestra "No se puede importar el tema", el JSON no cumple el esquema oficial. En este proyecto el problema fue que las líneas de cuadrícula (`gridline`) de los ejes estaban como objeto anidado; la forma válida es con propiedades planas (`gridlineShow`, `gridlineColor`). También conviene no usar un radio de borde de 9999 en los slicers.
2. Fuente: el tema pide Inter; si no está instalada, Power BI usa Segoe UI.
3. Archivo → Opciones → Datos actuales: desactivar "Detección automática de tablas de fechas" antes de crear la tabla Calendario manual.

## Fase 1 — Modelo de datos

Esquema en estrella: **Ventas** (hechos) relacionada con **Productos**, **Sucursales** y **Calendario**.

### 1.1 Tabla Calendario

```
Calendario =
CALENDAR(DATE(2026,1,1), DATE(2026,5,31))
```

Relación `Calendario[Date]` → `Ventas[Fecha]` (uno a varios) y marcar como tabla de fechas.

### 1.2 Medidas de negocio (las que se muestran en el dashboard)

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

Ventas por Día = DIVIDE([Ventas Netas], DISTINCTCOUNT(Ventas[Fecha]))
```

`Ventas por Día` es la medida que permite comparar meses de distinta duración.

### 1.3 Medidas de variación vs. mes anterior (en el modelo, no mostradas)

```
Ventas Netas MA = CALCULATE([Ventas Netas], DATEADD(Calendario[Date], -1, MONTH))
Δ Ventas Netas % = DIVIDE([Ventas Netas] - [Ventas Netas MA], [Ventas Netas MA])
```

Se definieron análogas para Ganancia Bruta, Margen (en puntos porcentuales), Ticket Promedio y Descuentos. No se muestran en las tarjetas porque, con meses de distinta duración y un mes incompleto, las caídas que mostrarían no son reales (ver README).

### 1.4 Pareto 80/20

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

### 1.5 Participación por canal y método de pago

```
% del Total (Metodo Pago) =
DIVIDE([Ventas Netas], CALCULATE([Ventas Netas], ALL(Ventas[MetodoPago])))

% del Total (Canal) =
DIVIDE([Ventas Netas], CALCULATE([Ventas Netas], ALL(Ventas[CanalVenta])))
```

Todas las medidas se organizan en la tabla auxiliar `_Medidas`.

## Fase 2 — Estructura de la página

De arriba hacia abajo:

1. **Encabezado:** tarjeta con "Cafetería ABC".
2. **Fila de contexto y filtros:** título "Dashboard Ejecutivo de Rendimiento" con subtítulo, y 4 slicers: **Date** (rango "Entre", 01/01/2026–31/05/2026), **Ciudad**, **Categoría** y **Sucursal** (lista desplegable).
3. **Fila de KPIs:** 6 tarjetas.
4. **Fila 2:** Pareto de productos (izquierda) y tabla de productos (derecha).
5. **Fila 3:** ventas por sucursal, distribución por canal y método de pago.

## Fase 3 — KPIs (6 tarjetas)

| Tarjeta | Medida | Formato |
|---|---|---|
| Ventas Netas | `[Ventas Netas]` | Moneda, en miles |
| Ganancia Bruta | `[Ganancia Bruta]` | Moneda, en miles |
| Margen % | `[Margen %]` | `0.0%` |
| Ticket Promedio | `[Ticket Promedio]` | Moneda con 2 decimales |
| Descuentos | `[Descuentos]` | Moneda, en miles |
| Ventas por Día | `[Ventas por Día]` | Moneda, en miles (2 decimales) |

## Fase 4 — Fila 2

- **Ventas Netas y % Acumulado por Producto:** gráfico de líneas y columnas agrupadas. Eje = `Productos[Producto]`, columnas = `[Ventas Netas]`, línea (eje secundario 0–100%) = `[% Acumulado]`. Sin filtro Top N: se muestran los 15 productos, ordenados por ventas netas descendente.
- **Tabla de productos:** Producto, Categoría, `[Unidades Vendidas]` y `[Margen %]`, ordenada por margen descendente, con fila de total. Sin líneas verticales, separador horizontal `#F0EEEC`.

## Fase 5 — Fila 3

- **Ventas Netas por Sucursal:** barras horizontales. Eje = `Sucursales[Sucursal]`, valor = `[Ventas Netas]`, etiquetas de datos **fuera del extremo** y en color oscuro para que se lean sobre fondo blanco.
- **Ventas Netas por CanalVenta:** anillo con etiquetas de valor y porcentaje.
- **Ventas Netas por MetodoPago:** una barra 100% apilada con el porcentaje de cada método.

## Fase 6 — Tema y detalles visuales

- El tema define paleta, tipografía, radio de esquina de 16 px en tarjetas, sombra suave y estilo de slicers. Se revisó visual por visual que no se hubieran sobrescrito formatos.
- Los slicers se ven correctamente solo si el tema define explícitamente el color del texto y un radio de borde válido.
- Ocultar el eje X del gráfico de sucursales (muestra "$0 mil – $100 mil" y no aporta, porque ya hay etiquetas de datos).

## Fase 7 — Interactividad

- Los slicers filtran todos los visuales.
- Editar interacciones para que el clic en un visual resalte (no filtre) a los demás.

## Fase 8 — Verificación

1. **Cifras contra los datos reales.** Se recalcularon desde `Ventas.csv`: Ventas Netas $602,106, Ganancia Bruta $346,643, Margen 57.6%, Ticket Promedio $120.4.
2. **Filtros.** Probar cada slicer y confirmar que todos los visuales reaccionan.
3. **Fidelidad al mockup.** Las cifras del mockup (`code.html`) son ilustrativas y no corresponden a los datos reales; solo se usó como referencia visual.

## Limitaciones conocidas

- Datos sintéticos (5,000 transacciones, enero a 25 de mayo de 2026).
- El margen es bruto: no incluye renta, nómina ni comisiones de delivery.
- Sin comparativas mensuales visibles, por la razón explicada arriba.
