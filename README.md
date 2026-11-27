# Red de Entidades Federativas de México

Visualización interactiva (D3 + p5.js) de la red de Estados, capitales y Pueblos Mágicos.
Es un sitio estático: no necesita build ni servidor propio.

## Estructura

```
RedMéxico/
├── index.html            # la visualización (página principal del sitio)
├── data/                 # CSV que se publican junto a la página
│   ├── Points.csv        # nodos (se cargan solos al abrir el sitio)
│   ├── Links.csv         # conexiones (se cargan solas al abrir el sitio)
│   ├── Coordenadas.csv   # latitud y longitud de cada nodo (Red geográfica)
│   ├── mapa_mexico.js    # contorno de los estados (mapa base)
│   └── ...               # matrices y grados (descargables)
├── docs/                 # contexto del proyecto (no hace falta publicarlo)
├── vercel.json           # configuración para Vercel
├── netlify.toml          # configuración para Netlify
├── .nojekyll             # para GitHub Pages
└── .gitignore
```

Al abrir el sitio publicado, `index.html` carga `data/Points.csv` y `data/Links.csv`
automáticamente. Los botones **Cargar nodos** y **Cargar conexiones** siguen funcionando
para usar otros archivos. Si abres `index.html` con doble clic (file://), la carga automática
no corre y se usan los botones, como antes.

Coordenadas: [GeoNames](https://www.geonames.org/) (CC BY 4.0). Mapa base: [Natural Earth](https://www.naturalearthdata.com/) (dominio público).

## Probar en local

```bash
cd RedMéxico
python3 -m http.server 8000
# abrir http://localhost:8000
```

## Publicar

- **Netlify:** arrastra la carpeta a https://app.netlify.com/drop, o conecta el repositorio (usa `netlify.toml`).
- **Vercel:** `npx vercel` desde la carpeta, o importa el repositorio (framework "Other", sin build).
- **GitHub Pages:** sube la carpeta a un repositorio y en Settings → Pages elige la rama `main` y la carpeta raíz.

Nota: el nombre de la carpeta lleva acento (`RedMéxico`); en algunos servicios conviene que el
repositorio se llame sin acento, por ejemplo `red-mexico`.
