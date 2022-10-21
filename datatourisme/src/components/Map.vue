<template>
  <div>
    <div id="map">
      <l-map
        :center="center"
        :zoom="zoom"
        @update:zoom="zoomUpdated"
        @update:center="centerUpdated"
      >
        <l-tile-layer :url="url" />
        <l-marker
          v-for="marker in markers"
          :key="marker.id"
          :lat-lng="marker.coordinates"
        >
          <l-icon ref="icon">
            <img class="marker-icon" :src="iconurl" />
          </l-icon>
        </l-marker>
      </l-map>
    </div>
    <div id="search">
      <fieldset>
        <legend>Vous voyez actuellement les {{this.type}} situés à {{this.city}}</legend>
        <input
          type="text"
          placeholder="Search for a location"
          v-model="city"
        />
        <input
          type="text"
          placeholder="Search for a type"
          v-model="type"
        />
        <button v-on:click="searchLocation">
          Rechercher
        </button>
      </fieldset>
    </div>
    <div id="indication">
      <p>Nombre de résultats : {{this.markers.length}}</p>
      <p>Une liste des types est disponible ici : <a href="https://www.datatourisme.fr/ontology/core/">https://www.datatourisme.fr/ontology/core/</a></p>
      <p>Par exemple, vous pouvez tester avec : Restaurant, Hotel, Beach</p>
    </div>
  </div>
</template>

<script>
import { LMap, LTileLayer, LMarker } from "vue2-leaflet";
import "leaflet/dist/leaflet.css";
import { Icon } from "leaflet";
require("isomorphic-fetch");

delete Icon.Default.prototype._getIconUrl;
Icon.Default.mergeOptions({
  iconRetinaUrl: require("leaflet/dist/images/marker-icon-2x.png"),
  iconUrl: require("leaflet/dist/images/marker-icon.png"),
  shadowUrl: require("leaflet/dist/images/marker-shadow.png"),
});

export default {
  components: {
    LMap,
    LTileLayer,
    LMarker,
  },
  data() {
    return {
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      center: [46.16508302168833, -1.1641140809069404],
      zoom: 12,
      markers: [],
      city: "",
      type: "",
    };
  },
  mounted() {
    this.city = "La Rochelle";
    this.type = "Restaurant";
    this.loadCoordinates(this.city, this.type);
  },
  methods: {
    zoomUpdated(zoom) {
      this.zoom = zoom;
    },
    centerUpdated(center) {
      this.center = center;
    },
    async loadCoordinates(city, type) {
      try {
        const poi = await fetch("http://localhost:8086/", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            query: `
            {
                poi(filters: [
                    {isLocatedAt: {schema_address: {schema_addressLocality: {_eq: "${city}"}}}}
                    {rdf_type: {_in: ["https://www.datatourisme.fr/ontology/core#${type}"]}}
                  ]) {
                        total
                        results {
                            _uri      
                            rdfs_label {
                                value
                            }
                            isLocatedAt {
                                schema_geo {
                                schema_latitude, schema_longitude
                            }
                        }
                    }
                }
            }`,
          }),
        })
          .then((response) => response.json())
          .then((data) => {
            return data.data.poi;
          });
        let list_of_markers = [];
        let counter = 1;
        poi.results.forEach((item) => {
          let lat = item.isLocatedAt[0].schema_geo[0].schema_latitude[0];
          let long = item.isLocatedAt[0].schema_geo[0].schema_longitude[0];
          let coordinates = [lat, long];
          list_of_markers.push({
            id: counter,
            coordinates: coordinates,
          });
          counter++;
        });
        this.markers = list_of_markers;
      } catch (e) {
        console.log("ERREUR : " + e);
      }
    },
    searchLocation() {
      this.loadCoordinates(this.city, this.type);
    },
  },
};
</script>

<style>
#map {
  background-color: red;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 90%;
}
#search  {
  position: absolute;
  bottom: 0;
}
#sentence {
  position: absolute;
  bottom: 0;
  margin-bottom: 10px;
}
#indication {
  position: absolute;
  bottom: 0;
  margin-bottom: -120px;
}
</style>