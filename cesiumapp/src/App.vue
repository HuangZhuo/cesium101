<script setup>
import * as Cesium from 'cesium';
import { onMounted } from 'vue';
onMounted(async () => {
  Cesium.ArcGisMapService.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIxNGE4YmE0ZC1mNDkxLTQyMWYtOTA5Zi0xODdiZjYyZWQ0ZWQiLCJpZCI6MjM0NjYwLCJpYXQiOjE3MjM2MTMyMzJ9.0Ytps_E_feRZQ5L0dS_iaC5ZmKsLctBSYEruYuyqJXE'

  /**
   * P2 Cesium.Viewer
   * P3 使用自定义 Map Server (Provider)
   * https://sandcastle.cesium.com/?src=ArcGIS%20MapServer.html&label=Beginner
   */
  const imgProvider = Cesium.ArcGisMapServerImageryProvider.fromUrl(
    'https://services.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer'
  )

  const viewer = new Cesium.Viewer('cesiumContainer', {
    // 隐藏图层选择器
    baseLayerPicker: false,
    // 设置图像服务器
    baseLayer: Cesium.ImageryLayer.fromProviderAsync(imgProvider),
    // 设置地形服务器
    terrainProvider: await Cesium.createWorldTerrainAsync({
      requestWaterMask: true,
      requestVertexNormals: true,
    })
  });

  /**
   * P4 摄像头位置和方向设置
   */
  viewer.camera.setView({
    // destination:Cesium.Cartesian3.fromDegrees(
    //   113.318977, 23.114155, 2000 // 广州塔
    // ),
    destination: new Cesium.Cartesian3(1332761, -4662399, 4137888), // NYC
    orientation: {
      heading: Cesium.Math.toRadians(0),
      pitch: Cesium.Math.toRadians(-45),
    }
  });

  /**
   * P5 加载建筑群并设置样式着色
   * https://sandcastle.cesium.com/?src=3D%20Tiles%20Feature%20Styling.html&label=Beginner
   */
  const osmBuildingsTileset = await Cesium.createOsmBuildingsAsync();
  viewer.scene.primitives.add(osmBuildingsTileset);

  // https://cesium.com/learn/cesiumjs/ref-doc/Cesium3DTileStyle.html?classFilter=Cesium3DTileStyle
  osmBuildingsTileset.style = new Cesium.Cesium3DTileStyle({
    defines: {
      material: "${feature['building:material']}",
    },
    color: {
      conditions: [
        ["${material} === null", "color('white')"],
        ["${material} === 'glass'", "color('skyblue', 0.5)"],
        ["${material} === 'concrete'", "color('grey')"],
        ["${material} === 'brick'", "color('indianred')"],
        ["${material} === 'stone'", "color('lightslategrey')"],
        ["${material} === 'metal'", "color('lightgrey')"],
        ["${material} === 'steel'", "color('lightsteelblue')"],
        ["true", "color('white')"], // This is the else case
      ],
    },
  });

  /**
   * P6 区域划分和渲染
   * https://sandcastle.cesium.com/?src=GeoJSON%20and%20TopoJSON.html&label=DataSources
   */

  // const geoJsonPromise = Cesium.GeoJsonDataSource.load(
  //   './assets/SampleData/ne_10m_us_states.topojson',
  //   {
  //     stroke: Cesium.Color.HOTPINK,
  //     fill: Cesium.Color.PINK.withAlpha(0.5),
  //     strokeWidth: 3,
  //   }
  // );
  // viewer.dataSources.add(geoJsonPromise);

  Cesium.GeoJsonDataSource.load(
    "./assets/SampleData/ne_10m_us_states.topojson"
  )
    .then(function (dataSource) {
      viewer.dataSources.add(dataSource);

      //Get the array of entities
      const entities = dataSource.entities.values;

      const colorHash = {};
      for (let i = 0; i < entities.length; i++) {
        //For each entity, create a random color based on the state name.
        //Some states have multiple entities, so we store the color in a
        //hash so that we use the same color for the entire state.
        const entity = entities[i];
        const name = entity.name;

        let color = colorHash[name];
        if (!color) {
          color = Cesium.Color.fromRandom({
            alpha: 1.0,
          });
          colorHash[name] = color;
        }

        //Set the polygon material to our random color.
        entity.polygon.material = color;
        //Remove the outlines.
        entity.polygon.outline = false;

        //Extrude the polygon based on the state's population.  Each entity
        //stores the properties for the GeoJSON feature it was created from
        //Since the population is a huge number, we divide by 50.
        entity.polygon.extrudedHeight = entity.properties.Population / 50.0;
      }
    })
    .catch(function (error) {
      //Display any errrors encountered while loading.
      window.alert(error);
    });

  /**
   * P7 KML
   * https://sandcastle.cesium.com/?src=KML.html
   */
  const kmlOptions = {
    camera: viewer.scene.camera,
    canvas: viewer.scene.canvas,
    screenOverlayContainer: viewer.container,
  }
  viewer.dataSources.add(
    Cesium.KmlDataSource.load(
      './assets/SampleData/facilities.kml',
      kmlOptions,
    )
  );

  /**
   * P8 CZML
   * https://sandcastle.cesium.com/?src=CZML.html&label=DataSources
   */
  viewer.dataSources.add(
    Cesium.CzmlDataSource.load('./assets/SampleData/Vehicle.czml')
  )
  viewer.scene.camera.setView({
    destination: Cesium.Cartesian3.fromDegrees(-116.52, 35.02, 95000),
    orientation: {
      heading: 6,
    },
  })

})


</script>

<template>
  <!-- <div>
    <a href="https://vitejs.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <HelloWorld msg="Vite + Vue" /> -->
  <div id="cesiumContainer"></div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>