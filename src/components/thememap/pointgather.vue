<template>
  <div class="com-pointgather">
    <div id="map"></div>

    <div class="control">
      <div style="padding-top: 10px">
        <el-button id="button1">取消聚类</el-button>
      </div>

      <div style="padding-top: 10px">
        <el-button id="button2">执行聚类</el-button>
      </div>
    </div>
  </div>
</template>
<script>
import companyimage from "../../assets/img/company.png";
import heatMapData from "../../assets/json/平均薪资热力图.json";
import testjson from "../../assets/json/point.json";
const mapboxgl = require("mapbox-gl");
export default {
  name: "pointgather",
  mounted() {
    this.initmap();
  },
  methods: {
    initmap() {
      this.$mapboxgl.accessToken =
        "pk.eyJ1IjoiY2hlbmpxIiwiYSI6ImNrcWFmdWt2bjBtZGsybmxjb29oYmRzZzEifQ.mnpiwx7_cBEyi8YiJiMRZg";
      var map = new this.$mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/chenjq/cl010ychv001214pdpa5xyq5a",
        center: [108.82, 33.17],
        zoom: 3.5,
      });

      document.getElementById("button2").addEventListener("click", () => {
        map.setLayoutProperty("points", "visibility", "none");
        map.setLayoutProperty("clusters", "visibility", "visible");
        map.setLayoutProperty("cluster-count", "visibility", "visible");
        map.setLayoutProperty("unclustered-point", "visibility", "visible");
        map.fitBounds([
          [90, 45], // 边界的西南角
          [120, 20], // 边界的东北角
        ]);
      });
      document.getElementById("button1").addEventListener("click", () => {
        map.setLayoutProperty("points", "visibility", "visible");
        map.setLayoutProperty("clusters", "visibility", "none");
        map.setLayoutProperty("cluster-count", "visibility", "none");
        map.setLayoutProperty("unclustered-point", "visibility", "none");
        map.fitBounds([
          [90, 25], // 边界的西南角
          [130, 45], // 边界的东北角
        ]);

        map.addLayer({
          id: "points",
          type: "symbol",
          source: "point1", // reference the data source
          layout: {
            "icon-image": "company", // reference the image
            "icon-size": 0.15,
          },
        });
      });
      map.on("load", function () {
        //从我们的GeoJSON数据中添加一个新的数据源，并设置
        // 'cluster'选项为true。GL-JS将向源数据添加point_count属性。
        map.addSource("sensicjson", {
          type: "geojson",
          //指向GeoJSON数据。这个例子显示了所有的M1.0+地震
          // 15年12月22日至16年1月21日。
          data: testjson,
          cluster: true,
          clusterMaxZoom: 14, // Max zoom to cluster points on
          clusterRadius: 50, //每个集群点的半径(默认为50)
        });
        //添加数据
        map.loadImage(companyimage, (error, image) => {
          if (error) throw error;

          // Add the image to the map style.
          map.addImage("company", image);
          map.addSource("point1", {
            type: "geojson",
            data: testjson,
          });
        });
        //添加圆形聚合图层
        map.addLayer({
          id: "clusters",
          type: "circle",
          source: "sensicjson",
          filter: ["has", "point_count"],
          paint: {
            //使用步骤表达式(https://docs.mapbox.com/mapbox-gl-js/style-spec/#expressions-step)
            //用三个步骤实现三种类型的循环:
            // *蓝色，20px圆，点计数小于100
            // *黄色，30px的圆圈，点计数在100和750之间
            // *粉红色，40px的圆圈，当点数大于等于750
            "circle-color": [
              "step",
              ["get", "point_count"],
              "#B0E0E6",
              100,
              "#87CEFA",
              750,
              "#00BFFF",
            ],
            "circle-radius": [
              "step",
              ["get", "point_count"],
              20,
              100,
              30,
              750,
              40,
            ],
          },
          //"source-layer": "button2"
        });

        //添加数字图层
        map.addLayer({
          id: "cluster-count",
          type: "symbol",
          source: "sensicjson",
          filter: ["has", "point_count"],
          paint: {
            "text-color": "#000000",
          },
          layout: {
            "text-field": "{point_count_abbreviated}",
            "text-font": ["DIN Offc Pro Medium", "Arial Unicode MS Bold"],
            "text-size": 12,
          },
          //"source-layer": "button2"
        });

        //添加未聚合图层
        map.addLayer({
          id: "unclustered-point",
          type: "circle",
          source: "sensicjson",
          filter: ["!", ["has", "point_count"]],
          paint: {
            "circle-color": "#11b4da",
            "circle-radius": 4,
            "circle-stroke-width": 1,
            "circle-stroke-color": "#fff",
          },
          //"source-layer": "button2"
        });

        map.on("load", () => {
          jsonCallback;
        });

        function jsonCallback(err, data) {
          if (err) {
            throw err;
          }
          data.features = data.features.map((d) => {
            d.properties.month = new Date(d.properties.time).getMonth();
            // d.properties.coordinates = new location(d.properties.geometry).getElementById("coordinates");
            return d;
          });
        }

        map.on("click", "unclustered-point", (e) => {
          // Copy coordinates array.
          const coordinates = e.features[0].geometry.coordinates.slice();
          const name = e.features[0].properties.name;

          // Ensure that if the map is zoomed out such that multiple
          // copies of the feature are visible, the popup appears
          // over the copy being pointed to.
          while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
            coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          }

          new mapboxgl.Popup().setLngLat(coordinates).setHTML(name).addTo(map);
        });
        // Change the cursor to a pointer when the mouse is over the places layer.
        map.on("mouseenter", "unclustered-point", () => {
          map.getCanvas().style.cursor = "pointer";
        });

        // Change it back to a pointer when it leaves.
        map.on("mouseleave", "unclustered-point", () => {
          map.getCanvas().style.cursor = "";
        });

        //检查集群单击（点击聚合图层地图级别中心点变化）
        map.on("click", "clusters", function (e) {
          var features = map.queryRenderedFeatures(e.point, {
            layers: ["clusters"],
          });
          var clusterId = features[0].properties.cluster_id;
          map
            .getSource("sensicjson")
            .getClusterExpansionZoom(clusterId, function (err, zoom) {
              if (err) return;
              map.easeTo({
                center: features[0].geometry.coordinates,
                zoom: zoom,
              });
            });
        });
      });
    },
  },
  // destroyed(){
  //   this.map.removeLayer("")
  // }
};
</script>

<style scoped lang="less">
#map {
  position: relative;
  width: 100%;
  height: 100%;
  z-index: 0;
}

.com-pointgather {
  position: relative;
  z-index: 10;
  height: 100%;
  width: 100%;
  background: transparent;
  .control {
    position: absolute;
    left: 64%;
    top: 47%;
    width: 14%;
    height: 38%;
    #button1 {
      z-index: 9999;
    }
    #button2 {
      z-index: 9999;
    }
  }
}

/deep/.el-button {
  background: url("../../assets/img/fq/wggl_tab.png");
  background-size: 100% 100%; // background: #24bff390;
  border: 0px solid #d80d4a;
  color: #ffffff;
  padding: 12px 20px;
  font-size: 14px;
  border-radius: 5px;
}

/deep/.el-button:focus,
.el-button:hover {
  color: #75f8ed;
  border-color: #c6e2ff; // background-color: #ecf5ff;
}

.mapboxgl-popup {
  max-width: 200px;
}
</style>