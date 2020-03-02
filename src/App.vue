<template>
  <div class="home row no-gutters">
    <div class="maskRule" :style="{transform : ruleScale}">
      <img src="./images/maskRule.jpg" alt="口罩購買規則">
      <button @click="ruleScale = 'scale(0)'">X</button>
    </div>
    <div class="col-md-3">
      <div v-if="cityName.length" class="toolbox">
        <div class="sticky-top text-white shadow-sm p-3 cityName">
          <h4>{{ today.year }}年{{ today.month }}月{{ today.day }}日 {{ daylist[today.whatday] }}</h4>
          <h5 class="ruleTitle"><a @click="ruleScale = 'scale(1)'">點我看口罩購買規則</a></h5>
          <div class="form-group d-flex">
            <select id="cityName" class="form-control mr-2 rounded-lg selCity"
            v-model="select.city" @change="select.area = ''">
              <option value="">-請選擇-</option>
              <option :value="c.CityName" v-for="c in cityName" :key="c.CityName">
                {{ c.CityName }}
              </option>
            </select>
            <select id="area" class="form-control ml-2 rounded-lg selCity" v-if="select.city.length"
              v-model="select.area" @change="updateSelect">
              <option value="">-請選擇-</option>
              <option :value="a.AreaName"
                v-for="a in cityName.find((city) => city.CityName === select.city).AreaList"
                :key="a.AreaName">
                {{ a.AreaName }}
              </option>
            </select>
          </div>
        </div>

        <ul class="list-group">
          <template v-for="(item, key) in data">
            <a class="list-group-item text-left mx-2 my-1 rounded-lg" :key="key"
              v-if="item.properties.county === select.city
                && item.properties.town === select.area"
              :class="{ 'highlight': item.properties.mask_adult || item.properties.mask_child}"
              @click="penTo(item)">
              <h5>{{ item.properties.name }}</h5>
              <h6><a :href="`https://www.google.com.tw/maps/place/${item.properties.address}`"
                target="_blank" title="Open By Google Map">
                {{ item.properties.address }}</a></h6>
              <h6>{{ item.properties.phone }}</h6>
              <h6>{{ item.properties.note }}</h6>
              <div class="maskAmount">
                <button class="btn btn-dark rounded-pill">
                成人口罩{{ item.properties.mask_adult}}
                </button>
                <button class="btn btn-info rounded-pill">
                兒童口罩{{ item.properties.mask_child}}
                </button>
              </div>
            </a>
          </template>
        </ul>
      </div>
    </div>
    <div class="col-md-9">
      <div id="map"></div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import L from 'leaflet';
import cityName from './assets/cityName.json';

let osmMap = {};

const iconsConfig = {
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
};
const icons = {
  green: new L.Icon({
    iconUrl: 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
    ...iconsConfig,
  }),
  grey: new L.Icon({
    iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-grey.png',
    ...iconsConfig,
  }),
};

// const markers = new L.MarkerClusterGroup().addTo(osmMap);
// osmMap.addLayer(markers);

const osm = {
  addMapMarker(x, y, item) {
    const icon = item.mask_adult || item.mask_child ? icons.green : icons.grey;
    L.marker([y, x], {
      icon,
    }).addTo(osmMap).bindPopup(`<h5>${item.name}</h5>
    <h6><a href="https://www.google.com.tw/maps/place/${item.address}" target="_blank">${item.address}</a></h6>
    <h6>${item.phone}</h6>
    <div class="maskAmount">
                <button class="btn btn-dark rounded-pill">
                成人${item.mask_adult}
                </button>
                <button class="btn btn-info rounded-pill">
                兒童${item.mask_child}
                </button>
              </div>
    <small>最後更新時間: ${item.updated}</small>`);
  },
  removeMapMarker() {
    osmMap.eachLayer((layer) => {
      if (layer instanceof L.Marker) {
        osmMap.removeLayer(layer);
      }
    });
  },
  penTo(x, y, item) {
    const icon = item.mask_adult || item.mask_child ? icons.green : icons.grey;
    osmMap.panTo(new L.LatLng(y, x));
    L.marker([y, x], {
      icon,
    }).addTo(osmMap).bindPopup(`<h5>${item.name}</h5>
    <h6><a href="https://www.google.com.tw/maps/place/${item.address}" target="_blank">${item.address}</a></h6>
    <h6>${item.phone}</h6>
    <div class="maskAmount">
                <button class="btn btn-dark rounded-pill">
                成人${item.mask_adult}
                </button>
                <button class="btn btn-info rounded-pill">
                兒童${item.mask_child}
                </button>
              </div>
    <small>最後更新時間: ${item.updated}</small>`).openPopup();
  },
};

const Today = new Date();

export default {
  name: 'home',
  data: () => ({
    cityName,
    data: {},
    osmMap: {},
    select: {
      city: '高雄市',
      area: '鼓山區',
    },
    today: {
      year: Today.getFullYear(),
      month: Today.getMonth() + 1,
      day: Today.getDate(),
      whatday: Today.getDay(),
    },
    daylist: ['星期日', '星期一', '星期二', ' 星期三', '星期四', '星期五', '星期六'],
    ruleScale: 'scale(0)',
  }),
  methods: {
    updateMarker() {
      const pharmacies = this.data.filter((pharmacy) => {
        if (!this.select.area) {
          return pharmacy.properties.county === this.select.city;
        }
        return pharmacy.properties.town === this.select.area;
      });
      pharmacies.forEach((pharmacy) => {
        const { properties, geometry } = pharmacy;
        osm.addMapMarker(
          geometry.coordinates[0],
          geometry.coordinates[1],
          properties,
        );
      });
      this.penTo(pharmacies[0]);
    },
    removeMarker() {
      osm.removeMapMarker();
    },
    penTo(pharmacy) {
      const { properties, geometry } = pharmacy;
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties);
    },
    updateSelect() {
      this.removeMarker();
      this.updateMarker();
      const pharmacy = this.data.find((item) => item.properties.town === this.select.area);
      const { geometry, properties } = pharmacy;
      osm.penTo(geometry.coordinates[0], geometry.coordinates[1], properties);
    },
  },
  mounted() {
    // OSM
    osmMap = L.map('map', {
      center: [25.03, 121.55],
      zoom: 15,
    });

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '<a target="_blank" href="https://www.openstreetmap.org/">© OpenStreetMap 貢獻者</a>',
      maxZoom: 18,
    }).addTo(osmMap);

    this.$http.get('https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json')
      .then((res) => {
        this.data = res.data.features;
        this.updateMarker();
      });
  },
};
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';
@import url("https://fonts.googleapis.com/css?family=Noto+Sans+TC&display=swap");

* {
  font-family: 'Noto Sans TC', sans-serif;
}

#map {
  height: 100vh;
}

.home {
  height: 100vh;
  position: relative;
}

.highlight {
  background: #ddfffa;
}

.toolbox {
  height: 100vh;
}

.cityName {
  height: 25%;
  background-color: rgb(108, 203, 206);
}

.selCity,
.selCity option,
.selCity:focus {
  color: #a0a0a0;
}

.selCity:focus {
  outline: none;
}

.list-group {
  height: 75%;
  overflow-y: auto;
  a {
    cursor: pointer;
    text-decoration: none;
  }
}

.maskAmount {
  display: flex;
  button {
    width: 50%;
    box-sizing: border-box;
    margin: 3px;
  }
}

.maskAmount button {
  color: #fff;
}

.ruleTitle{
  text-decoration: underline;
  a:hover{
    color: rgb(82, 133, 133);
    cursor: pointer;
  }
}

.maskRule {
  width: 100%;
  height: 100vh;
  background-color: rgba(0, 0, 0, .7);
  position: absolute;
  left: 0;
  top: 0;
  z-index: 1030;
  transition: .5s;
  :focus{
    outline: none;
  }
  img {
    width: 400px;
    height: 600px;
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    margin: auto;
  }
  button {
    width: 50px;
    height: 50px;
    font-size: 48px;
    color: #fff;
    background: transparent;
    border: none;
    position: absolute;
    right: 0;
    top: 0;
  }
}

@media screen and (max-width: 768px) {
  .col-md-3{
    height: 25%;
    .toolbox {
      height: 100%;
      .cityName {
        height: 100%;
      }
      .list-group {
        height: 0;
      }
    }
  }
  .col-md-9{
    height: 75%;
    #map {
      height: 100%;
    }
  }
}
</style>
