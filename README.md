# 🌍 Análisis de deuda pública mundial | Power BI (2025)

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black) ![Power Query](https://img.shields.io/badge/Transformación-Power%20Query-236A8A) ![Estado](https://img.shields.io/badge/Estado-Dashboard%20desarrollado-376B50)

## 📌 Descripción

Análisis de la **deuda pública y el producto interior bruto (PIB) por países**, a partir de tablas públicas de Datosmacro consultadas para **2025**. El proyecto incluye la importación de datos web, su limpieza con Power Query, la construcción de un modelo relacional y un dashboard interactivo en Power BI Desktop.

La relación entre las tablas permite comparar el PIB y la deuda de un mismo país. El indicador **deuda pública / PIB** muestra el peso de la deuda respecto al tamaño de su economía. Esta relación de datos **no implica por sí misma una correlación estadística ni una relación causal**.

## 🛠️ Herramientas

- **Power BI Desktop:** modelo, medidas e informe.
- **Power Query (lenguaje M):** importación, limpieza y transformación.
- **DAX:** medidas para los indicadores agregados.
- **GitHub:** documentación y presentación del proyecto.

## 📂 Estructura del repositorio

```text
powerbi-analisis-deuda-mundial/
├── README.md
├── PowerBI/
│   └── Reto_01_Analisis_Deuda_Mundial_2025.pbix
└── Images/
    └── dashboard_deuda_mundial_2025.png
```

> El archivo `.pbix` y la captura ya están guardados localmente. Esta estructura organiza su publicación en GitHub.

## 🔗 Fuentes de datos

- [Deuda pública por países — Datosmacro](https://datosmacro.expansion.com/deuda)
- [PIB por países — Datosmacro](https://datosmacro.expansion.com/pib)

Las tablas se importaron mediante **Obtener datos → Web**, con acceso anónimo, y se prepararon en **Transformar datos**. El análisis se ha trabajado con las tablas seleccionadas de **2025**; al proceder de páginas web que pueden actualizarse, los resultados reflejan la información disponible en el momento de la importación.

## 🧹 Preparación y limpieza en Power Query

### Deuda pública

1. Selección de la tabla de deuda pública y eliminación de columnas no necesarias (incluidas fechas no utilizadas en el análisis).
2. Revisión del historial de **Pasos aplicados**, teniendo en cuenta que modificar pasos intermedios puede afectar a los posteriores.
3. Limpieza de `Países`, eliminando el indicador `[+]` y espacios sobrantes mediante **Reemplazar valores**.
4. Limpieza de `Deuda per Capita`, quitando los separadores `.` y el símbolo `€` antes de convertirla de texto a número.
5. Comprobación de los tipos de datos y asignación del nombre **Deuda Pública** a la consulta.

### PIB

1. Previsualización y selección de la tabla de **PIB anual**.
2. Eliminación de columnas vacías o no necesarias.
3. Limpieza de `PIB anual`, retirando `.` y `M€` antes de convertir la columna a tipo numérico.
4. Normalización de `Países`, retirando `[+]` y espacios.
5. Revisión de **Vista → Calidad de columna**, uso temporal de filtros para investigar filas anómalas y eliminación posterior de esos filtros.
6. Eliminación de una fila no válida mediante **Quitar filas → Quitar errores**, según la limpieza realizada.

Se evitó equiparar `null` con cero: un dato ausente no significa que el importe sea cero. Tras revisar ambas consultas, se utilizó **Cerrar y aplicar**.

## 🔍 Incidencia resuelta: cuatro países duplicados

Al relacionar las tablas por `Países`, Power BI mostró inicialmente una cardinalidad **varios a uno (`*:1`)**. Para investigar la causa, se agruparon los registros de deuda por país y se contó cuántas filas había en cada grupo.

Se identificaron **cuatro países con dos registros**: Líbano, Eritrea, Myanmar y Venezuela. Las filas no eran idénticas: una contenía deuda total y deuda sobre PIB; la otra, deuda per cápita. Eliminar duplicados directamente habría hecho perder información.

### Solución

Se utilizó **Agrupar por → Países → Todas las filas**, creando la columna auxiliar `Datos`. A continuación se recuperó el primer valor no nulo de cada campo mediante tres columnas personalizadas en M:

```powerquery
// Deuda total (M€)
List.First(List.RemoveNulls([Datos][#"Deuda total (M.€)"]), null)

// Deuda sobre PIB
List.First(List.RemoveNulls([Datos][#"Deuda (%PIB)"]), null)

// Deuda per cápita
List.First(List.RemoveNulls([Datos][#"Deuda Per Cápita"]), null)
```

`List.RemoveNulls` descarta los valores nulos del grupo y `List.First` recupera el primero disponible. El argumento `null` evita un error cuando no hay ningún valor. Después se eliminó `Datos` y se asignaron los tipos finales: texto para país, entero para deuda total y deuda per cápita, y porcentaje para deuda / PIB.

### Validación de registros

| País | Deuda total (M€) | Deuda / PIB | Deuda per cápita |
|---|---:|---:|---:|
| Líbano | 59.873 | 357,45 % | 7.372 |
| Eritrea | 4.610 | 260,39 % | 1.010 |
| Myanmar | 25.299 | 37,63 % | 522 |
| Venezuela | 42.242 | 30,72 % | 872 |

La tabla de deuda resultante mostraba **192 filas y 4 columnas**, con una fila por país. Se conservaron los valores ausentes de origen, como los campos de deuda no disponibles para Yemen.

## 🔄 Modelo de datos

Se relacionaron **Deuda Pública** y **PIB** por el campo `Países`:

| Propiedad | Configuración final |
|---|---|
| Campo de unión | `Países` |
| Cardinalidad | **Uno a uno (`1:1`)** |
| Relación activa | Sí |
| Dirección de filtro cruzado | Ambas |

Esta configuración corresponde a las claves únicas observadas tras la limpieza. La relación permite cruzar los indicadores de deuda y PIB por país, pero no demuestra una asociación estadística entre sus valores.

## 📊 Medidas y dashboard

Se creó una tabla auxiliar **`Medidas creadas`** mediante **Introducir datos**, para organizar las medidas DAX separadamente de las tablas importadas. El dashboard contiene:

- **Tres tarjetas KPI:** deuda pública total, media del porcentaje de deuda / PIB y PIB total.
- **Dos gráficos de rectángulos (treemaps):** deuda pública total por país y PIB total por país.
- **Un mapa de áreas rellenas:** países geolocalizados, con `Países` en **Ubicación** y la medida `DeudaTotal` en **Información sobre herramientas**.

Durante la configuración del mapa inicialmente solo se mostraba China: se corrigió al trasladar `Países` de **Leyenda** a **Ubicación**. Para utilizar los mapas se revisaron las opciones de seguridad de los elementos visuales geográficos en Power BI.

### 🎨 Diseño del informe

Se personalizó el dashboard con **azul marino**, texto blanco, esquinas redondeadas, emojis en los KPI y una distribución de tres tarjetas superiores, dos treemaps y un mapa inferior. También se ajustaron los fondos y títulos para dar uniformidad a los visuales.

**Valores mostrados en la última captura del informe:** deuda total `99 mill.`, deuda media / PIB `61,06 %` y PIB total `127,72 mill.`. Estas cifras son las mostradas por Power BI; **antes de publicarlas como resultados económicos deben comprobarse las unidades y la agregación de las medidas DAX**.

> **Control de calidad pendiente:** el treemap de PIB presenta una categoría **`(En blanco)`** con un peso considerable. Es necesario revisar los países sin correspondencia o los valores ausentes antes de extraer conclusiones del gráfico. Por ello, no se presentan aún conclusiones cuantitativas definitivas.

### 🖼️ Captura del dashboard

![Dashboard de deuda pública mundial 2025](Images/dashboard_deuda_mundial_2025.png)

> Para que se vea en GitHub, colocar la captura en `Images/dashboard_deuda_mundial_2025.png`.

## 💡 Aprendizajes

- Seleccionar e importar tablas HTML desde una web.
- Limpiar caracteres y unidades monetarias antes de convertir tipos.
- Comprender la importancia del orden de los **Pasos aplicados**.
- Diferenciar errores, registros duplicados, datos complementarios y valores `null`.
- Utilizar **Agrupar por**, listas y columnas personalizadas en M sin perder información.
- Interpretar cardinalidades y relaciones entre tablas.
- Organizar medidas DAX y configurar KPI, treemaps y mapas.
- Revisar la calidad de los datos antes de interpretar resultados visuales.

## 🚀 Próximos pasos

Estas son posibles mejoras y comprobaciones posteriores; no impiden conservar el trabajo ya realizado.


- [ ] Revisar la categoría `(En blanco)` del treemap de PIB.
- [ ] Validar las fórmulas DAX, las unidades monetarias y los totales mostrados.
- [x] Guardar una captura del dashboard; pendiente incorporarla a la carpeta `Images/` del repositorio.
- [ ] Documentar las expresiones DAX exactas cuando estén verificadas.
- [ ] Añadir conclusiones cuantitativas basadas en datos validados.
- [ ] Publicar el archivo `.pbix`, la imagen y este README en GitHub.

## 👤 Autoría y contexto

**Angelo Herrera** · [GitHub: aherrera180](https://github.com/aherrera180)

Proyecto práctico de aprendizaje realizado durante el módulo de introducción a Power BI de **ThePower**. Las transformaciones y las incidencias descritas documentan el trabajo realizado durante el reto guiado.
