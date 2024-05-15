<template>
    <div>
        <div class="map">
            <l-map style="height: 100vh; width: 100vw" ref="map" :zoom="defaultZoom" :minZoom="minZoom"
                :center="currentCenter" :options="{ zoomControl: false }" @zoomend="showZoomedPopup"
                @click="clickOutsideMarkers" class="map">
                <l-tile-layer :url="url" :attribution="attribution" />
                <l-control-zoom position="topright"></l-control-zoom>
                <l-geo-json v-show="show" :geojson="geojson" :options-style="styleFunction" />
                <vue2-leaflet-marker-cluster ref="myCluster" :options="clusterOptions">
                    <l-marker v-for="library in libraries" :key=library.id :id="`mark-${library.id}`" ref="mapMarkers"
                        :lat-lng="[library.latitude, library.longitude]" rise-on-hover:true :icon="icon" @click="clickMarker(library.library_code)">
                        <l-tooltip v-show="showNames" :options="{ permanent: false, interactive: true }">
                            {{ library.name }}
                        </l-tooltip>
                        <l-popup :options="{ permanent: true, interactive: true, autoclose: false }">
                            <div>
                                <b>{{ library.name }}</b>
                                <br>
                                <template v-if="library.opening_hours">
                                    <br>
                                    <b>Opening Hours:</b> {{ library.opening_hours }}
                                    <br>
                                </template>
                                <br>
                                <b>Address:</b>
                                <template v-if="library.address_line_1">
                                    <br>
                                    {{ library.address_line_1 }}
                                </template>
                                <template v-if="library.address_line_2">
                                    <br>
                                    {{ library.address_line_2 }}
                                </template>
                                <template v-if="library.address_line_3">
                                    <br>
                                    {{ library.address_line_3 }}
                                </template>
                                <template v-if="library.address_line_4">
                                    <br>
                                    {{ library.address_line_4 }}
                                </template>
                                <template v-if="library.street">
                                    <br>
                                    {{ library.street }}
                                </template>
                                <template v-if="library.postal_code">
                                    <br>
                                    {{ library.postal_code }}
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
                                <br>
                                <template v-if="library.website">
                                    <br>
                                    <b>Website:</b>
                                    <br>
                                    <a :href="library.website" target="_blank" class="cropped-link">{{ library.website
                                        }}</a>
                                </template>
                                <br>
                                <template v-if="library.notes">
                                    <br>
                                    <b>Additional Notes:</b>
                                    <br>
                                    <t class="add-notes" v-html="library.notes" />
                                    <br>
                                </template>
                            </div>
                        </l-popup>
                    </l-marker>
                </vue2-leaflet-marker-cluster>
            </l-map>
        </div>

        <div class="search">
            <search-bar :libraries="this.libraries" @search-library="searchLibrary" :search-input="searchText" />
        </div>
    </div>
</template>

<script>
import { icon, DivIcon, Point } from "leaflet";
import { LMap, LTileLayer, LMarker, LGeoJson, LPopup, LControlZoom, LTooltip } from "vue2-leaflet";
import Vue2LeafletMarkerCluster from 'vue2-leaflet-markercluster'
import SearchBar from "./SearchBar.vue";

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
        Vue2LeafletMarkerCluster,
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
                iconUrl: require("@/assets/marker-icon.png"),
                iconSize: [32, 37],
                iconAnchor: [16, 37]
            }),
            clusterClicked: false,
            zoomMarker: null,
            libraries: null,
            selectedLibrary: null,
            fillColor: "#4485CA",
            url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
            attribution:
                '&copy; Map data from <a href="http://osm.org/copyright">OpenStreetMap</a> contributors, ',

            clusterOptions: {
                maxClusterRadius: 40,
                animate: false,
                iconCreateFunction: cluster => {
                    var childCount = cluster.getChildCount();
                    var sizeConstant = 5;
                    var c = " my-marker-cluster-size-";
                    if (childCount < 2) {
                        sizeConstant = 22;
                        c += "1";
                    } else if (childCount < 5) {
                        c += "2";
                        sizeConstant = 25;
                    } else if (childCount < 10) {
                        c += "3";
                        sizeConstant = 27;
                    } else if (childCount < 15) {
                        c += "4";
                        sizeConstant = 30;
                    } else if (childCount < 50) {
                        c += "5";
                        sizeConstant = 35;
                    } else {
                        c += "6";
                        sizeConstant = 40;
                    }

                    return new DivIcon({
                        html: "<div class='cluster-content'><span>" + childCount + "</span></div>",
                        className: "marker-cluster" + c,
                        iconSize: new Point(sizeConstant, sizeConstant)
                    });
                }
            },
        };
    },
    computed: {
        styleFunction() {
            return () => {
                return {
                    weight: 4,
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
        this.loading = false;
        this.currentCenter = this.defaultCenter;

        try {
            const resp = await fetch(
                process.env.VUE_APP_DIRECTORIES_BASE_URL + "api/v1/libraries",
                {
                    method: 'GET',
                    headers: {
                        'Accept': 'application/json',
                        'Authentication': process.env.VUE_APP_DIRECTORIES_API_KEY
                    }
                });

            // turn the libraries into a json array without the libraries that have no coordinates
            this.libraries = (await resp.json()).data.filter(l => l.latitude !== null && l.longitude !== null);
            this.handleQueryParams();
        } catch (error) {
            console.log(
                "Error fetching and parsing data"
            );
        }
    },
    methods: {
        searchLibrary (searchResult) {
            this.selectedLibrary = searchResult;
            this.$refs.map.mapObject.flyToBounds([[searchResult.latitude, searchResult.longitude]]);
            this.$nextTick().then(() => {
                const markerToOpen = this.$refs.mapMarkers.find(m => {
                    m.$el.id === `mark-${searchResult.id}`;
                    return m.$el.id === `mark-${searchResult.id}`;
                });
                this.zoomMarker = markerToOpen.mapObject;
            });
            this.changeQueryParams(searchResult.library_code);
        },
        showZoomedPopup() {
            this.currentZoomLevel = this.$refs.map.mapObject.getZoom();
            if (this.zoomMarker) {
                this.$nextTick(() => {
                    this.zoomMarker.openPopup();
                    this.zoomMarker = undefined;
                });
            }
        },
        clickMarker(id) {
            this.changeQueryParams(id)
        },
        clickOutsideMarkers() {
            this.selectedLibrary = "";
        },
        handleQueryParams() {
            const params = new URLSearchParams(window.location.search);
            const library = params.get('library');
            if (!library) {
                return;
            }
            this.selectedLibrary = this.libraries.find(l => l.library_code === library);
            try {
                this.searchLibrary(this.selectedLibrary);
            } catch (error) {
                console.log(error);
            }
        },
        changeQueryParams(id) {
            const params = new URLSearchParams();
            params.set('library', id);

            // Check if the library query parameter already exists
            const existingLibraryParam = params.get('library');

            // If it doesn't exist, append it to the URL
            if (!existingLibraryParam) {
                window.history.replaceState({}, '', `${window.location.pathname}?${params}`);
            } else {
                // If it exists, simply replace the existing query parameter
                window.history.replaceState({}, '', `?${params}`);
            }
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
    filter: grayscale(0%);
}

.add-notes {
    white-space: pre-wrap;
}

.search {
    position: absolute;
    top: 25px;
    left: 50px;
    z-index: 1000;
}

.cropped-link {
    display: inline-block;
    width: 100%;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

/* Override defaults from circle to square */
.marker-cluster {
    border-radius: 50%;
    background-clip: padding-box;
    outline: 4px solid #4E4A99;
}

.my-marker-cluster-size-1 {
    background-color: #8D92CD;
}

.my-marker-cluster-size-2 {
    background-color: #5E5AB1;
}

.my-marker-cluster-size-3 {
    background-color: #5E5AB1;
}

.my-marker-cluster-size-4 {
    background-color: #504A96;
}

.my-marker-cluster-size-5 {
    background-color: #4B4585;
}

.my-marker-cluster-size-6 {
    background-color: #0C0056;
}

.cluster-content {
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 1.2em;
    font-weight: bold;
    color: white;
}
</style>