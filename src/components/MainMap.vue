<script>
    import { onMounted } from 'vue';
    import maplibregl from 'maplibre-gl';

    export default {
        setup() {
            onMounted(() => {
                const map = new maplibregl.Map({
                    container: 'map',
                    style: 'https://api.maptiler.com/maps/streets/style.json?key=cXh6kpqsIV9qrsbI2IKs',
                    center: [44.15207, 18.40307], // starting position [lng, lat],
                    zoom: 3.7 // starting zoom
                });
                new maplibregl.Marker()
                    .setLngLat([44.48904, 11.49439])
                    .addTo(map);
            });
        },
        data() {
            return {

            }
        },
    }
</script>

<template>
    <div id="map" style="width: 100%; height: 50vh;"></div>
</template>

<style scoped lang="scss">

</style>