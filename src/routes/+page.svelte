<script lang="ts">
  import maplibregl from "maplibre-gl";
  import "maplibre-gl/dist/maplibre-gl.css";
  import { onMount } from "svelte";

  const origMaxHeight = 3000;
  const makeColorRelief = (
    maxHeight: number
  ): maplibregl.ExpressionSpecification => {
    const scale = maxHeight / origMaxHeight;

    // prettier-ignore
    return [
      "interpolate",
      ["linear"],
      ["elevation"],
      // original code: https://github.com/NathanMOlson/NathanMOlson.github.io/blob/9fccba3165751dea4253c38c389e99494e43e83a/color_relief/index.html#L46-L59
      //
      // 地理院標高タイルの換算は、10倍して10,000を引く
      // cf. https://zenn.dev/shi_works/articles/6f50506d5bc3b0
         0   * scale * 10 - 10000, "#c0e0ffff",
         0.1 * scale * 10 - 10000, "#d8d8d8ff",
         3   * scale * 10 - 10000, "#548359ff",
       255   * scale * 10 - 10000, "#32482dff",
      1000   * scale * 10 - 10000, "#32482dff",
      1200   * scale * 10 - 10000, "#adb7a3ff",
      1700   * scale * 10 - 10000, "#bfbfb8ff",
      1750   * scale * 10 - 10000, "#e6f4fdff",
      3000   * scale * 10 - 10000, "#ffffffff",
      // nodata (補間されてしまうのでエッジの表示がおかしいが、とりあえずそれっぽくなる)
      828860.7, "#ffffffff",  // (2^23 - 1) * 0.1 - 10000
      828860.8, "#00000000",  //  2^23      * 0.1 - 10000
      // 標高0m以下 (ほんとはちゃんとグラデーションさせた方がいいが、とりあえず 0m と同じ色にする)
      828860.9, "#c0e0ffff",  // (2^23 + 1) * 0.1 - 10000
    ];
  };

  let map: maplibregl.Map;

  let scale = $state(origMaxHeight);
  $effect(() => {
    scale;
    console.log(map);
    if (map) {
      map.setPaintProperty(
        "color_relief",
        "color-relief-color",
        makeColorRelief(scale)
      );
    }
  });

  onMount(() => {
    map = new maplibregl.Map({
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
            maxzoom: 14,
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
              "color-relief-color": makeColorRelief(origMaxHeight),
            },
          },
        ],
      },
    });
  });
</script>

<div style="height: 4vh">
  最大標高：
  <label>
    <input
      type="number"
      bind:value={scale}
      min="1"
      max={origMaxHeight * 2}
      step="1"
    />
    <input
      type="range"
      bind:value={scale}
      min="1"
      max={origMaxHeight * 2}
      step="1"
    />
  </label>
</div>
<div id="map" style="height: 96vh"></div>
