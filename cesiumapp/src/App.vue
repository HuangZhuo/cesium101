<script setup>
import * as Cesium from 'cesium';
import { onMounted } from 'vue';
onMounted(async () => {
  Cesium.ArcGisMapService.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIxNGE4YmE0ZC1mNDkxLTQyMWYtOTA5Zi0xODdiZjYyZWQ0ZWQiLCJpZCI6MjM0NjYwLCJpYXQiOjE3MjM2MTMyMzJ9.0Ytps_E_feRZQ5L0dS_iaC5ZmKsLctBSYEruYuyqJXE'

  const imgProvider = Cesium.ArcGisMapServerImageryProvider.fromUrl(
    'https://services.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer'
  )

  const viewer = new Cesium.Viewer('cesiumContainer', {
    // 隐藏图层选择器
    baseLayerPicker: false,
    // 设置图像服务器
    // https://sandcastle.cesium.com/?src=ArcGIS%20MapServer.html&label=Beginner
    baseLayer: Cesium.ImageryLayer.fromProviderAsync(imgProvider),
    // 设置地形服务器
    terrainProvider: await Cesium.createWorldTerrainAsync({
      requestWaterMask: true,
      requestVertexNormals: true,
    })
  });

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

  // 加载建筑群
  // https://sandcastle.cesium.com/?src=3D%20Tiles%20Feature%20Styling.html&label=Beginner
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