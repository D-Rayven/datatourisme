<template>
  <l-map
        :center="center"
        :zoom="zoom"
        class="map"
        ref="map"
        @update:zoom="zoomUpdated"
        @update:center="centerUpdated"    
    >
        <l-tile-layer
            :url="url"
        >
    </l-tile-layer>
    <l-marker
        v-for="marker in markers"
        :key="marker.id"
        :lat-lng="marker.coordinates"
    >
        <l-icon ref="icon">
            <img class="marker-icon" :src="iconurl"/>
        </l-icon>
    </l-marker>
  </l-map>
</template>

<script>
import { LMap, LTileLayer, LMarker } from 'vue2-leaflet';
import 'leaflet/dist/leaflet.css';
import {Icon} from "leaflet";
require('isomorphic-fetch');

delete Icon.Default.prototype._getIconUrl;
Icon.Default.mergeOptions({
  iconRetinaUrl: require("leaflet/dist/images/marker-icon-2x.png"),
  iconUrl: require("leaflet/dist/images/marker-icon.png"),
  shadowUrl: require("leaflet/dist/images/marker-shadow.png")
});

export default {
    components: {
        LMap,
        LTileLayer,
        LMarker
    },
    data() {
        return {
            url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
            center: [46.16508302168833, -1.1641140809069404],
            zoom: 12,
            markers: [],
        }
    },
    mounted() {
        this.loadCoordinates();
    },
    methods: {
        zoomUpdated(zoom) {
            this.zoom = zoom;
            console.log(this.markers);
        },
        centerUpdated(center) {
            this.center = center;
        },
        async loadCoordinates() {
            try {
                const poi = await fetch('http://localhost:8086/', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    query: `
                    {
                        poi(filters: [
                            {isLocatedAt: {schema_address: {schema_addressLocality: {_eq: "La Rochelle"}}}}]) {
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
                    }`
                })
                }).then(response => response.json())
                .then(data => {
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
                    coordinates: coordinates
                });
                counter++;
                });
                this.markers = list_of_markers;
            } catch(e) {
                console.log("L'erreur est ici : " + e)
            }    
        },
    }
}
</script>

<style>
.map {
  position: absolute;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
</style>