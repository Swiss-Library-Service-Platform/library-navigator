<template>
    <div>
        <div class="map">
            <l-map  
                style="height: 100vh; width: 100vw" 
                ref="map" 
                :zoom="defaultZoom"
                :minZoom="minZoom"
                :center="currentCenter" 
                :options="{ zoomControl: false }" 
                @zoomend="showZoomedPopup"
                @click="clickOutsideMarkers"
            >
                <l-tile-layer :url="url" :attribution="attribution" />
                <l-control-zoom position="topright"></l-control-zoom>
                <l-geo-json v-show="show" :geojson="geojson" :options-style="styleFunction"/>
                <l-marker
                    v-for="library in libraries"
                    :key=library.id
                    :id="`mark-${library.id}`"
                    ref="mapMarkers"
                    :lat-lng="[library.latitude, library.longitude]"
                    rise-on-hover:true
                    @click="selectLibrary(library)">
                    <l-tooltip
                        v-show="showNames"
                        :options="{ permanent: false, interactive: true }">
                        {{ library.name }}
                    </l-tooltip>
                    <l-popup :options="{ permanent: true, interactive: true }">
                        <div>
                            <b>{{ library.name }}</b>
                            <br>
                            <template v-if="library.opening_hours">
                                <br>
                                <b>Opening Hours:</b> {{ library.opening_hours  }}
                                <br>
                            </template>
                            <br>
                            <b>Address:</b>
                            <template v-if="library.address_line_1">
                                <br>
                                {{ library.address_line_1  }}
                            </template>
                            <template v-if="library.address_line_2">
                                <br>
                                {{ library.address_line_2  }}
                            </template>
                            <template v-if="library.address_line_3">
                                <br>
                                {{ library.address_line_3  }}
                            </template>
                            <template v-if="library.address_line_4">
                                <br>
                                {{ library.address_line_4  }}
                            </template>
                            <template v-if="library.street">
                                <br>
                                {{ library.street  }}
                            </template>
                            <template v-if="library.postal_code">
                                <br>
                                {{ library.postal_code  }}
                                {{ library.city }}
                            </template>
                            <br>
                            <br>
                            <b>Services:</b>
                            <template v-if="library.service_bursar != 0">
                                <br>
                                <span style="margin-left: 20px;">Bursar</span>
                            </template>
                            <template v-if="library.service_courier != 0">
                                <br>
                                <span style="margin-left: 20px;">Courier</span>
                            </template>
                            <template v-if="library.service_slspweb_selfcheck != 0">
                                <br>
                                <span style="margin-left: 20px;">Webselfcheck</span>
                            </template>
                            <template v-if="library.service_personal_delivery != 0">
                                <br>
                                <span style="margin-left: 20px;">Personal Delivery</span>
                            </template>
                            <template v-if="library.service_digitization_requests != 0">
                                <br>
                                <span style="margin-left: 20px;">Digitization Requests</span>
                            </template>
                            <template v-if="library.website">
                                <br>
                                <b>Website:</b>
                                <br>
                                <a :href="library.website" target="_blank">{{ library.website }}</a>
                            </template>
                            </div>
                    </l-popup>
                </l-marker>
            </l-map>
        </div>
        <div class="search">
            <search-bar
                :libraries="this.libraries"
                @search-library="selectLibrary"
                @search-reset="resetMap" 
                :search-input="searchText"/>
        </div>
    </div>
</template>

<script>
import { Icon, icon } from "leaflet";
import { LMap, LTileLayer, LMarker, LGeoJson, LPopup, LControlZoom, LTooltip } from "vue2-leaflet";
//import libraryJson from '../assets/prod.json';
import SearchBar from "./SearchBar.vue";

delete Icon.Default.prototype._getIconUrl;
Icon.Default.mergeOptions({
    iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
    iconUrl: require('leaflet/dist/images/marker-icon.png'),
    shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
});

export default {
    name: "MapComponent",
    components: {
        LMap,
        LTileLayer,
        LGeoJson,
        LControlZoom,
        LMarker,
        LPopup,
        LTooltip,
        SearchBar,
    },
    data() {
        return {
            loading: false,
            show: true,
            showNames: true,
            enableTooltip: true,
            defaultZoom: 8,
            currentZoomLevel: 8,
            minZoom: 8,
            defaultCenter: [46.8182, 8.2275],
            currentCenter: [46.8182, 8.2275],
            geojson: null,
            icon: icon({
                iconUrl: "src/assets/marker-icon.png",
                iconSize: [32, 37],
                iconAnchor: [16, 37]
            }),
            libraries: null,
            selectedLibrary: null,
            fillColor: "#4485CA",
            url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
            attribution:
                //    '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
                'Swiss Library Service Platform, <a href="https://slsp.ch/en">SLSP</a>',
        };
    },
    computed: {
        styleFunction() {
            return () => {
                return {
                    weight: 10,
                    color: "#4E4A99",
                    opacity: 2,
                    fillOpacity: 0.000
                };
            };
        },
        searchText() {
            if (this.selectedLibrary == null) {
                return "";
            }
            return this.selectedLibrary.name;
        },
    },
    async created() {
        this.loading = true;
        const response = await fetch("https://raw.githubusercontent.com/ZHB/switzerland-geojson/master/country/switzerland.geojson")
        const data = await response.json();
        this.geojson = data;
        //this.libraries = libraryJson;
        this.loading = false;
        this.currentCenter = this.defaultCenter;

        const librariesJson = await fetch("http://localhost:3008/api/libraries");
       // this.libraries = await librariesJson.json();
        const allLibraries = await librariesJson.json();
        this.libraries = allLibraries.filter(library => library.latitude !== null || library.latitude !== null);

    },
    methods: {
        selectLibrary(searchResult) {
            this.selectedLibrary = searchResult;
            this.$refs.map.mapObject.flyToBounds([[searchResult.latitude, searchResult.longitude]]);
            const markerToOpen = this.$refs.mapMarkers.find(m => {
                return m.$el.id === `mark-${searchResult.id}`;
            });
            this.zoomMarker = markerToOpen.mapObject;
        },
        showZoomedPopup() {
            this.currentZoomLevel  = this.$refs.map.mapObject.getZoom();
            if (this.zoomMarker) {
                this.$nextTick(() => {
                    this.zoomMarker.openPopup();
                    this.zoomMarker = undefined;
                });
            }
        },
        updateZoomLevel() {
            this.currentZoomLevel = this.$refs.map.mapObject.getZoom();
        },
        clickOutsideMarkers() {
            this.selectedLibrary = "";
        },
        resetMap() {
            this.$nextTick(() => {
                this.currentCenter = this.defaultCenter;
            })
        },
    }
};
</script>
<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

.map {
    position: relative;
    height: 100wh;
    width: 100vw;
}

.search {
    position: absolute;
    top: 25px;
    left: 50px;
    z-index: 1000;
}
</style>