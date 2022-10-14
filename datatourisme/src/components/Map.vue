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
            markers: [
                {id: 1, coordinates: [ 46.16226442018983, -1.1488847613775959 ]},
                {id: 2, coordinates: [ 46.169464863196154, -1.1263651677337871 ]},
                {id: 3, coordinates: [ 49.102160, 6.158850 ]},
                {id: 4, coordinates: [ 49.136010, 6.199630 ]},
                {id: 5, coordinates: [ 49.105563, 6.182234 ]},
            ]
        }
    },
    methods: {
        zoomUpdated(zoom) {
            this.zoom = zoom;
            console.log(this.markers);
        },
        centerUpdated(center) {
            this.center = center;
        }
        }
    }
</script>

<style>
    .map {
        position: absolute;
        width: 100%;
        height: 100%;
        overflow: hidden
    }
</style>