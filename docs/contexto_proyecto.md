# Contexto del proyecto: Red de Entidades Federativas

## Objetivo
Representar como red las entidades de México: cada nodo es un Estado, una capital o un Pueblo Mágico. Una conexión indica relación entre entidades y su peso es la distancia (km).

## Datos de entrada (CSV)
- **Points** (nodos): 2 columnas, `Capital` (nombre único) y `ENTIDAD` (categoría). 238 filas. No tiene peso de nodo.
  - Categorías: `Estados` (33, incluye San Luis Potosí), `Capital` (31, nombres con sufijo `(C)`) y `Pueblo Mágico` (174).
- **Links** (conexiones): 3 columnas (origen, destino, distancia). La primera fila viene vacía (`,,`) y se trata como encabezado. 295 filas. No tiene columnas Tipo ni Sentido.
  - Las distancias llevan texto (`215 km`, `180km`) o solo número (`50`). Las conexiones de pueblo→estado y capital→estado usan 50 como valor nominal.

## Problemas detectados en los datos
- **Nombres de Links corregidos en el CSV:** las rutas entre ciudades apuntan a las capitales `(C)` (`CDMX` → `Ciudad de México (C)`); `Taxcala` → `Tlaxcala`; `Zactaecas` → `Zacatecas`. `Celaya` y `León` se agregaron a Points (categoría `Ciudad`) y a Coordenadas. Resultado: 240 nodos y 282 conexiones.
- `Yucatan` se empareja con `Yucatán` (sin acentos). Los demás no se corrigen: se avisan.
- 13 conexiones duplicadas (mismo par en ambos sentidos); se conserva la primera.
- `Guerrero` e `Hidalgo` aparecen en Links como destino de pueblos (`Guerrero,Coahuila`; `Hidalgo,Guanajuato`) y coinciden con los estados del mismo nombre; la ambigüedad no se resuelve.

## Entregables
1. `matriz_adyacencia.csv`: matriz binaria simétrica, 240 × 240 (primera versión, con corrección manual de nombres y Celaya/León como nodos extra).
2. `matriz_adyacencia_ponderada_km.csv`: igual, con km en lugar de 1.
3. `grados_por_nodo.csv`: nodo, tipo, grado y suma de km, ordenado por grado. Mayores: Edo Mex (22), Puebla (19), Jalisco (18), Veracruz (17).
4. `index.html`: visualización interactiva (ver abajo).
5. Este archivo de contexto.

Nota: las matrices y los grados se calcularon con una normalización de nombres distinta a la del HTML (el HTML no corrige nombres y los avisa), así que sus cifras no coinciden exactamente.

## Visualización HTML
- Un solo archivo, funciona en el navegador sin servidor. Carga D3 v7.8.5 y p5.js v1.9.0 desde cdnjs (requiere internet la primera vez).
- Interfaz en español. El usuario carga los CSV con los botones **Cargar nodos** y **Cargar conexiones**.
- **Librería (opción c):** D3 lee CSV, calcula escalas y posiciones; p5.js (modo instancia) dibuja en canvas. Zoom/paneo con `d3.zoom`.
- **Modos (cuatro):**
  - *Agrupación por Área*: `d3.pack` + fuerzas, contorno convexo suavizado por categoría.
  - *Bipartito en columnas*: dos columnas elegibles, curvas bezier, orden por baricentro. Por defecto Estados (izquierda) y Pueblo Mágico (derecha).
  - *Red geográfica*: cada nodo en su latitud y longitud (`data/Coordenadas.csv`, fuente GeoNames, CC BY 4.0) sobre el contorno de los estados (`data/mapa_mexico.js`, Natural Earth). Los estados usan el punto representativo de GeoNames. `Amatlán de Cañas · Compostela` se ubicó en Amatlán de Cañas.
  - *Arranque centralizado*: un hub por nodo de la categoría elegida (por defecto Estados), vecinos en anillos; los vecinos compartidos se repiten en cada hub.
- **Estilo elegido:** Nunito (si está instalada), fondo neutro `#f4f3ef`, paleta de verdes (Estados `#0f4d2e`, Capital `#2e9e5b`, Pueblo Mágico `#8fcf6b`), enlaces con degradado origen→destino y grosor medio proporcional al peso, nodos medianos (radio 5–18 según número de conexiones), etiquetas siempre visibles (con omisión de solapes) y tooltip al pasar el cursor, **sin flechas** (capa opcional apagada) y partículas opcionales apagadas.
- **Funciones:** barra lateral colapsable con leyenda y filtros por categoría, casilla para nodos sin conexiones (ocultos por defecto), panel Capas, tooltips con los encabezados del CSV, clic para resaltar nodo y vecinos, clic en fondo para deseleccionar, avisos visibles de lectura.
- **Reglas de CSV:** UTF-8 con o sin BOM, separador coma o punto y coma, comillas dobles, recorte de espacios, filas vacías ignoradas; columnas leídas por posición. Columnas opcionales de Links (peso, tipo, sentido) se detectan; Tipo y Sentido están implementados pero no se usan con estos datos.
- **Estructura del código:** 1 Configuración, 2 Estado, 3 Lectura de CSV, 4 Modelo de datos, 5 Escalas, 6 Modos, 7 Interacción, 8 Interfaz, 9 Inicio. Constantes editables marcadas con `// AJUSTABLE:`.

## Estado
- El código pasó revisión de sintaxis y una prueba en Node con los CSV reales (parser, modelo, layouts y dibujo con contexto simulado). No se ha probado en un navegador real.
- Pendientes posibles: decidir el tratamiento de `Guerrero`/`Hidalgo`, y añadir más modos del diccionario original (Red, Radial, Concéntrico, Edge bundling, Radial segmentado, Cuerdas, Arco, Hive, Red ego) si se necesitan.
