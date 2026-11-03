# Why the maps showed “API key required”

Both pages, `conferenze_prototipo_11_04.html` and `conferenze_prototipo_05_05.html`, are self-contained Leaflet maps exported from R. Each one draws its background with a single tile layer:

```text
addProviderTiles("CartoDB.Positron", ..., { maxZoom: 18 })
```

`CartoDB.Positron` is the light-gray CARTO style. Leaflet builds each tile URL from the provider definition already embedded in the page:

```text
https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png
```

Those URLs still respond with HTTP 200 and a small PNG. The image is no longer a map. It is a placeholder that reads “API KEY REQUIRED” and points to `carto.com/basemaps/apikey`. CARTO stopped serving this public raster style without an API key. Because the response is a successful image, Leaflet paints it instead of showing a network error, so the map frame, markers, and controls stay in place while the background is the placeholder.

## The change

The tile provider on both pages is now Esri’s light-gray canvas, which still serves tiles without a key and keeps the same pale look:

```text
addProviderTiles("Esri.WorldGrayCanvas", ..., { maxZoom: 18, maxNativeZoom: 16 })
```

`Esri.WorldGrayCanvas` is already defined in the Leaflet providers script in each file. It requests tiles from:

```text
https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}
```

Esri only publishes native tiles for this style through zoom 16. The map itself still allows zoom 18, so `maxNativeZoom: 16` tells Leaflet to reuse the zoom-16 tiles and scale them when the user zooms closer. Without that, zooms 17 and 18 would request tiles that do not exist.

Markers, the reset-view button, and the layer control were left as they were. Only the basemap call changed.

## How it was checked

A local server served both HTML files. In the browser, each map loaded 256×256 tiles from the Esri URL above, showed country labels and the conference markers, and still zoomed. The “API key required” placeholder was gone.
