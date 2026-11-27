# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Interactive network visualization of Mexico's states, state capitals and Pueblos Mágicos. It is a **static site with no build step, no package.json and no test suite**: everything lives in the single file `index.html` (inline CSS + JS), which loads D3 v7.8.5 and p5.js v1.9.0 from cdnjs. The UI, code identifiers and comments are in Spanish; keep new code and UI text in Spanish to match.

`docs/contexto_proyecto.md` is the detailed project brief (data issues, design decisions, pending work). Read it before non-trivial changes.

## Running locally

```bash
python3 -m http.server 8000   # from the repo root, then open http://localhost:8000
```

Served over http(s), `index.html` auto-loads `data/Points.csv` and `data/Links.csv` (`CSV_PREDETERMINADOS` / `cargarDatosPredeterminados`). Opened via `file://`, auto-load is skipped on purpose and the user loads CSVs with the **Cargar nodos** / **Cargar conexiones** buttons — both paths must keep working.

Deployment targets: Netlify (`netlify.toml`, publish `.`), Vercel (`vercel.json`, `.vercelignore` excludes `docs/` and `README.md`), GitHub Pages (`.nojekyll`). There is no build command anywhere.

## Architecture of `index.html`

The script is divided into numbered sections marked with `// ====` banners: 1 Configuración, 2 Estado, 3 Lectura de CSV, 4 Modelo de datos, 5 Escalas, 6 Modos, 7 Interacción, 8 Interfaz, 9 Inicio. User-tunable constants are tagged `// AJUSTABLE:` — prefer changing those over hardcoding values elsewhere.

Data flow:
1. **CSV reading** (`parsearCSV`): auto-detects `,` vs `;`, handles BOM, trims cells, reads columns **by position**, not by header name. `leerNumero` extracts numbers from strings like `215 km`.
2. **Model** (`construirModelo` → global `M`): nodes `{id, name, cat, deg, nb, size, ...}` from Points (col 0 = name, col 1 = category); links from Links (origin, destination, optional weight/type/direction). Link endpoints are matched to nodes exactly, then by accent/case-insensitive `normalizar`; unmatched names, duplicates and self-loops are dropped and reported as visible warnings (`#avisos`) rather than silently fixed.
3. **Scales** (`crearEscalas` → global `ESC`): node radius by degree, link width by weight.
4. **Modes**: each mode is registered in both `MODOS` (id + display name) and `MODOS_IMPL` (`{layout, dibujar}`). `layout(vis, W, H)` takes the filtered visible graph (`calcularVisibles`) and returns a scene `{insts, links, titulos, limites, ...}`; `dibujar(ctx, k, scene, frame)` draws it onto the p5 canvas 2D context. Shared drawing primitives (`dibujarEnlaces`, `dibujarNodos`, `dibujarEtiquetas`, `dibujarBase`, `geoLinea`, `geoBez`) live in section 6.0. Errors in one mode are caught in `actualizarEscena` and shown in `#error-modo` without breaking other modes. Scene "instances" are separate from model nodes because a node can appear multiple times (e.g. shared neighbors repeated per hub in *Arranque centralizado*).
5. **Rendering/interaction**: p5 in instance mode only draws; D3 handles CSV, scales, layouts (`d3.pack`, forces, `packSiblings`) and zoom/pan (`d3.zoom`). Redraw is on-demand via `state.sucio` (continuous only when the particles layer is on). All UI state is in the single `state` object; `reconstruir()` rebuilds model, scales and side panel after data changes.

Current modes are only *Agrupación por Área*, *Bipartito en columnas* (default columns in `BIPARTITO_PREDETERMINADO`: Estados left, Pueblo Mágico right; the shorter column is sorted by degree, the other by neighbor position), *Arranque centralizado* and *Red geográfica*; other modes listed in the brief are pending, not missing by accident.

## Data (`data/`)

- `Points.csv` and `Links.csv` are the live inputs. `Links.csv` has an empty first row (`,,`) that acts as the header. `Celaya` and `León` are nodes with category `Ciudad` (appended at the end so category colors, assigned by first appearance, don't shift). Don't "fix" names in the parser — the design is to warn, and corrections belong in the CSV.
- `Coordenadas.csv` (`Nodo,Latitud,Longitud,...`) feeds *Red geográfica*; names must match Points (typos included). It is optional state (`state.coords`), auto-loaded over http(s) and loaded with **Cargar coordenadas** under `file://`. Values come from GeoNames (CC BY 4.0); state nodes use GeoNames' representative point.
- `mapa_mexico.js` is the base map (simplified Natural Earth outlines). It is a `<script>` rather than fetched JSON so it also works under `file://`. It is drawn by projecting vertices manually (no `d3.geoPath`, so ring winding doesn't matter).
- A scene with `pantalla: true` (the geographic one) keeps node radius and link width constant in screen pixels while zooming.
- `matriz_adyacencia*.csv` and `grados_por_nodo.csv` are precomputed downloadables generated with a different name normalization than the HTML, so their numbers don't exactly match what the page shows. They are not read by `index.html`.

## Visual style constraints (chosen by the user)

Green palette by category (Estados `#0f4d2e`, Capital `#2e9e5b`, Pueblo Mágico `#8fcf6b`), background `#f4f3ef`, Nunito font, gradient links, labels always visible with overlap avoidance, arrows and particles layers off by default (`CAPAS_INICIALES`).
