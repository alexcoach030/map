<template>
  <div>
    <p v-if="!calculated">Производим расчеты...</p>
    <p v-if="calculated">Общая дистанция: {{ distance.toFixed(2) }} метров</p>
    <h3>Main map</h3>
    <div ref="mapContainer" class="l-map"></div>
  </div>
</template>

<script>
//Для карты использовал vue-plugin leaflet
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import geo from "../../geo.json"
export default {
  data() {
    return {
      text: "Add RUSSIA",
      data: Object.assign({}, geo),
      map: null,
      mapLayer: null,
      url: "https://waadsu.com/api/russia.geo.json",
      lon1: null,
      lon2: null,
      lat1: null,
      lat2: null,
      distance: 0,
      calculated: false
    };
  },
  methods: {
    addRussia() {
      //Получаем данные GeoJSON по ссылке
      fetch(this.url, {headers: {'Access-Control-Allow-Origin': '*',
          'Content-Type': 'application/json',}, mode: 'no-cors'})
          .then((response) => {return response.json();})
          .then((data) => {
            this.data = Object.assign({}, data);
            //Метод для добавление новых данных
            this.addGeoJson(this.data);
            //Метод для подсчета дистанции
            this.countDistance(this.data);
            //вывод данных
            this.text = "distance =" + this.distance.toFixed(2) + "m";
          })
          .catch((error) => {
            console.log(error);
          });
    },
    createMapInstance() {
      this.map = L.map(this.$refs.mapContainer, {
        //Минимальный зум
        minZoom: 3,
      }).setView([55.7558, 37.6173], 10); // Начальный координаты (Москва)
      this.mapLayer = L.tileLayer(
          // Добавление тайлов с сайта (тоже бесплатно и без ограничений, можно даже было убрать копирайты)
          "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
          {
            //чтобы карта не повторялась,
            noWrap: true,
            attribution:
            // сам копирайт
                '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
          }
      );
      //добавление самой конфигурации тайлов к главной карте
      this.map.addLayer(this.mapLayer);
    },
    addGeoJson(GeoJSON) {
      //Добавление geoJSON данных на карту
      L.geoJSON(GeoJSON).addTo(this.map);
    },
    countDistance(data) {
      // Радиус кривизны планеты (средний)
      const R = 6371e3;
      if (data) {
        this.distance = 0;
        let a = data.features[0].geometry.coordinates;
        for (let i = 0; i < a.length; i++) {
          //итерация по объекту
          for (let j = 0; j < a[i].length; j++) {
            // точки 1,2,3..
            // console.log(a[i][j]);
            for (let k = 0; k < a[i][j].length; k++) {
              // a[i][j][k][0] - широта/longitude
              // a[i][j][k][1] - долгота/latitude
              this.lon1 = a[i][j][k][0];
              this.lat1 = a[i][j][k][1];
              this.lon2 = a[i][j][a[i][j].length - 1][0];
              this.lat2 = a[i][j][a[i][j].length - 1][1];
              // угол в радианы переводятся
              const angle1 = (this.lat1 * Math.PI) / 180;
              const angle2 = (this.lat2 * Math.PI) / 180;
              // изменение угла в радианах
              const angulardiff1 = ((this.lat2 - this.lat1) * Math.PI) / 180;
              const angulardiff2 = ((this.lon2 - this.lon1) * Math.PI) / 180;
              const z =
                  Math.sin(angulardiff1 / 2) * Math.sin(angulardiff1 / 2) +
                  Math.cos(angle1) *
                  Math.cos(angle2) *
                  Math.sin(angulardiff2 / 2) *
                  Math.sin(angulardiff2 / 2);
              //угловое расстояние
              const c = 2 * Math.atan2(Math.sqrt(z), Math.sqrt(1 - z));
              const d = R * c; // Дистанция между двумя точками
              this.distance += d;
            }
          }
        }
      }
    },
  },
  // Монтируем при запуске приложения карту и маршрут
  mounted() {
    this.createMapInstance();
    /*this.addRussia();*///Закомментированный метод фетча, поскольку cors теперь на сервере не дает работать
    this.addGeoJson(this.data);
    //Метод для подсчета дистанции
    this.countDistance(this.data);
    //вывод данных
    this.text = "distance =" + this.distance.toFixed(2) + "m";
    this.calculated = true;
  },
};
</script>

<style scoped>
.l-map {
  width: 100%;
  height: 800px;
}
</style>