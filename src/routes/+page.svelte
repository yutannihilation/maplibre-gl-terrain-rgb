<script lang="ts">
  import maplibregl from "maplibre-gl";
  import "maplibre-gl/dist/maplibre-gl.css";
  import { onMount } from "svelte";

  onMount(() => {
    const map = new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        center: [139.6917, 35.6895],
        zoom: 8,
        sources: {
          terrain: {
            type: "raster-dem",
            tiles: [
              "https://cyberjapandata.gsi.go.jp/xyz/dem_png/{z}/{x}/{y}.png",
            ],
            tileSize: 256,
          },
          pale: {
            type: "raster",
            tiles: [
              "https://cyberjapandata.gsi.go.jp/xyz/pale/{z}/{x}/{y}.png",
            ],
            maxzoom: 18,
            tileSize: 256,
            attribution:
              '<a href="https://maps.gsi.go.jp/development/ichiran.html">地理院タイル</a>',
          },
        },
        layers: [
          {
            id: "pale",
            source: "pale",
            type: "raster",
          },
          {
            id: "color_relief",
            type: "color-relief",
            source: "terrain",
            paint: {
              "color-relief-opacity": 0.8,
              // prettier-ignore
              "color-relief-color": [
                "interpolate",
                ["linear"],
                ["elevation"],
                // 地理院標高タイルの換算は、10倍して10,000を引く
                // cf. https://zenn.dev/shi_works/articles/6f50506d5bc3b0
                -10000, "#c0e0ffff",
                -9999, "#d8d8d8ff",
                -9970, "#548359ff",
                -7450, "#32482dff",
                0, "#32482dff",
                2000, "#adb7a3ff",
                7000, "#bfbfb8ff",
                7500, "#e6f4fdff",
                20000, "#ffffffff"
              ],
            },
          },
        ],
      },
    });
  });
</script>

<div id="map" style="height: 100vh"></div>
