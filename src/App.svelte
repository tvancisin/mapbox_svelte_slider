<script>
  import MapboxCompare from "mapbox-gl-compare";
  import mapboxgl from "mapbox-gl";
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import { getCSV, getGeo } from "./utils";
  let height, width, map;
  let csv_path = ["./data/survey.csv"];
  let geojson_path = ["./data/unique_prio_grids.json"];
  let survey_data = [];
  let geo_data = [];
  const idp_camp_ids = [
    136504, 138664, 135782, 142255, 140817, 140816, 142980, 143704,
  ];

  import "mapbox-gl/dist/mapbox-gl.css";
  import "mapbox-gl-compare/dist/mapbox-gl-compare.css";

  let beforeMapContainer;
  let afterMapContainer;
  let comparisonContainer;

  onMount(async () => {
    mapboxgl.accessToken =
      "pk.eyJ1Ijoic2FzaGFnYXJpYmFsZHkiLCJhIjoiY2xyajRlczBlMDhqMTJpcXF3dHJhdTVsNyJ9.P_6mX_qbcbxLDS1o_SxpFg";
    mapboxgl.Compare = MapboxCompare;

    const beforeMap = new mapboxgl.Map({
      attributionControl: false,
      container: beforeMapContainer,
      style: "mapbox://styles/sashagaribaldy/cm4aigyc400qc01s6bzp76jjf",
      center: [31.307, 6.877],
      zoom: 5,
      maxZoom: 8,
      projection: "naturalEarth",
    });

    const afterMap = new mapboxgl.Map({
      attributionControl: false,
      container: afterMapContainer,
      style: "mapbox://styles/sashagaribaldy/cm0az6qe200pf01phd16v6qm0",
      center: [31.307, 6.877],
      zoom: 5,
      maxZoom: 8,
      projection: "naturalEarth",
    });

    // wait for both maps to load
    await Promise.all([
      new Promise((resolve) => beforeMap.on("load", resolve)),
      new Promise((resolve) => afterMap.on("load", resolve)),
    ]);

    new MapboxCompare(beforeMap, afterMap, comparisonContainer, {});

    // load CSV and GeoJSON
    const csv = await getCSV(csv_path);
    survey_data = csv[0];

    const geo = await getGeo(geojson_path);
    geo_data = geo[0];

    console.log("Data loaded:", survey_data, geo_data);
    // Filtered GeoJSON for beforeMap: only IDPs
    const geoBefore = {
      ...geo_data,
      features: geo_data.features.filter((f) =>
        idp_camp_ids.includes(f.properties.gid),
      ),
    };

    // Filtered GeoJSON for afterMap: exclude IDPs
    const geoAfter = {
      ...geo_data,
      features: geo_data.features.filter(
        (f) => !idp_camp_ids.includes(f.properties.gid),
      ),
    };

    [beforeMap, afterMap].forEach((m) => {
      const isBefore = m === beforeMap;
      const sourceData = isBefore ? geoBefore : geoAfter;
      const layerId = isBefore ? "polygons_fill_before" : "polygons_fill_after";

      if (!m.getSource("polygons")) {
        m.addSource("polygons", {
          type: "geojson",
          data: sourceData,
          generateId: true,
        });

        m.addLayer({
          id: layerId,
          type: "fill",
          source: "polygons",
          paint: {
            "fill-color": isBefore ? "steelblue" : "orange", // different colors
            "fill-opacity": isBefore ? 0.7 : 0.5, // different opacity
          },
        });
      }
    });
  });
</script>

<div id="comparison-container" bind:this={comparisonContainer}>
  <div id="before" bind:this={beforeMapContainer}></div>
  <div id="after" bind:this={afterMapContainer}></div>
</div>

<style>
  #comparison-container {
    height: 100vh;
    width: 100%;
    position: relative;
  }
  #before,
  #after {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
  }
</style>
