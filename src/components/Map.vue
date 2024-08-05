<template>
    <div class="map-wrap">
        <div class="map" ref="mapContainer"></div>
    </div>
</template>

<script setup>
    import { Map, MapStyle, Marker, config } from '@maptiler/sdk';
    import { shallowRef, onMounted, onUnmounted, markRaw } from 'vue';
    import '@maptiler/sdk/dist/maptiler-sdk.css';

    const mapContainer = shallowRef(null);
    const map = shallowRef(null);

    onMounted(() => {
        config.apiKey = 'cXh6kpqsIV9qrsbI2IKs';

        const initialState = { lng: 21.05688, lat: 41.35505, zoom: 4.2 };

        map.value = markRaw(new Map({
            container: mapContainer.value,
            style: MapStyle.STREETS,
            center: [initialState.lng, initialState.lat],
            zoom: initialState.zoom
        }));
        new Marker({ color: "#FF0000" })
            .setLngLat([11.34337, 44.49377])
            .addTo(map.value);
    }),
        onUnmounted(() => {
            map.value?.remove();
        })
</script>

<style scoped>
    .map-wrap {
        position: relative;
        width: 100%;
        height: calc(100vh - 77px);
    }

    .map {
        position: absolute;
        width: 100%;
        height: 100%;
    }
</style>