<script>
    import {FormField, HandlesValidationErrors} from 'laravel-nova';
    import L from "leaflet";
    import {LCircle, LMap, LMarker, LTileLayer} from 'vue2-leaflet';

    delete L.Icon.Default.prototype._getIconUrl;
    L.Icon.Default.mergeOptions({
        iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
        iconUrl: require('leaflet/dist/images/marker-icon.png'),
        shadowUrl: require('leaflet/dist/images/marker-shadow.png'),
    });

    export default {
        components: {
            LMap,
            LMarker,
            LTileLayer,
            LCircle,
        },

        mixins: [FormField, HandlesValidationErrors],

        props: ['resourceName', 'resourceId', 'field'],

        data: function () {
            return {
                tileUrl: 'https://{s}.tile.osm.org/{z}/{x}/{y}.png',
                mapOptions: {
                    boxZoom: false,
                    doubleClickZoom: 'center',
                    dragging: false,
                    scrollWheelZoom: 'center',
                    touchZoom: 'center',
                },
                markerOptions: {
                    interactive: false,
                },
            };
        },

        created: function () {
            if (this.field.tileProvider !== undefined) {
                this.tileUrl = this.field.tileProvider;
            }
        },

        computed: {
            defaultZoom: function () {
                return this.field.defaultZoom || 12;
            },

            locationIsSet: function () {
                if (this.value.latitude === undefined) {
                    this.setInitialValue();
                }

                return this.value.latitude > 0
                    || this.value.longitude > 0;
            },

            locationIsNotSet: function () {
                return !this.locationIsSet;
            },

            mapCenter: function () {
                if (this.value.latitude === undefined) {
                    this.setInitialValue();
                }

                return [
                    this.value.latitude,
                    this.value.longitude,
                ];
            },

            radiusIsSet: function () {
                return this.field.circleRadius !== undefined && this.field.circleRadius >= 0;
            },

            circleRadiusValue: function () {
                return this.field.circleRadius || 0;
            }
        },

        methods: {
            fill: function (formData) {
                formData.append(this.field.latitude, this.value.latitude);
                formData.append(this.field.longitude, this.value.longitude);
            },

            handleChange: function (value) {
                this.value.latitude = value.latitude;
                this.value.longitude = value.longitude;
            },

            mapMoved: function (event) {
                let coordinates = event.target.getCenter();

                this.value.latitude = coordinates.lat;
                this.value.longitude = coordinates.lng;
            },

            setInitialValue: function () {
                this.value = {
                    latitude: this.field.value[this.field.latitude || "latitude"] || 0,
                    longitude: this.field.value[this.field.longitude || "longitude"] || 0,
                };
            },
        },
    };
</script>

<template>
    <panel-item :field="field">
        <div slot="value">
            <span
                v-if="locationIsNotSet"
            >
                &#8212;
            </span>
            <l-map
                v-if="locationIsSet"
                class="z-10 map-field w-full form-control form-input-bordered overflow-hidden relative"
                ref="map"
                :center="mapCenter"
                :options="mapOptions"
                :zoom="defaultZoom"
                @move="mapMoved"
            >
                <l-tile-layer
                    :url="tileUrl"
                ></l-tile-layer>
                <l-marker
                    :options="markerOptions"
                    :lat-lng="mapCenter"
                ></l-marker>
                <l-circle
                    v-if="radiusIsSet"
                    :lat-lng="mapCenter"
                    :radius="circleRadiusValue"
                />
            </l-map>
        </div>
    </panel-item>
</template>

<style lang="scss" scoped>
    @import "~leaflet/dist/leaflet.css";

    .leaflet-pane {
        position: relative;
    }

    .map-field {
        height: 350px;
    }

    .leaflet-pane .leaflet-shadow-pane {
        display: none;
    }
</style>
