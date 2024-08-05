<script setup>
    import { ref, onMounted, onUnmounted, markRaw } from 'vue';
    import { Map, MapStyle, Marker, config } from '@maptiler/sdk';
    import '@maptiler/sdk/dist/maptiler-sdk.css';
    import axios from 'axios';

    const mapContainer = ref(null);
    const map = ref(null);

    const form = ref({
        title: '',
        description: ''
    });

    onMounted(() => {
        config.apiKey = 'cXh6kpqsIV9qrsbI2IKs';

        const initialState = { lng: 18.05688, lat: 42.35505, zoom: 5.2 };

        map.value = markRaw(new Map({
            container: mapContainer.value,
            style: MapStyle.STREETS,
            center: [initialState.lng, initialState.lat],
            zoom: initialState.zoom
        }));

        new Marker({ color: "#FF0000" })
            .setLngLat([11.34337, 44.49377])
            .addTo(map.value);
    });

    onUnmounted(() => {
        map.value?.remove();
    });

    const save_form = async () => {
        const address = form.title;
        // if (!form.title) {
        //     console.log('inserisci indirizzo');
        //     return;
        // }

        try {
            const response = await axios.get(`https://api.tomtom.com/search/2/geocode/${encodeURIComponent(address)}.json?key=SvIJjQs1GXAO5L5EXzIoRElBWyGEsSX2`);
            const { lat, lon } = response.data.result[0].position;
            console.log(response);

            localStorage.setItem('coordinates', JSON.stringify({ lat, lon }));
            console.log(lat, lon);

        } catch (error) {
            console.error('fallito')
        }
    }

    const getStoredCoordinates = () => {
        const coordinates = localStorage.getItem('coordinates');
        return coordinates ? JSON.parse(coordinates) : null;
    };
</script>

<template>
    <div class="map-wrap">
        <!-- Button trigger modal -->
        <button type="button" class="btn btn-primary ms-btn" data-bs-toggle="modal" data-bs-target="#exampleModal">
            Aggiungi Località
        </button>

        <!-- Modal -->
        <div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h1 class="modal-title fs-5" id="exampleModalLabel">Informazioni località</h1>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                    </div>
                    <div class="modal-body">
                        <form>
                            <label for="title">Indirizzo Località</label>
                            <input type="text" autocomplete="off" v-model="form.title" class="form-control" id="title">
                            <label for="description">Descrizione</label>
                            <textarea id="description" class="form-control" maxlength="500" rows="3"
                                v-model="form.description" autocomplete="off"></textarea>
                        </form>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Chiudi</button>
                        <button type="button" class="btn btn-primary" @click="save_form">Salva</button>
                    </div>
                </div>
            </div>
        </div>
        <div class="map" ref="mapContainer"></div>
    </div>
</template>

<style scoped lang="scss">
    .map-wrap {
        position: relative;
        width: 100%;
        height: calc(100vh - 56px);

        .ms-btn {
            position: absolute;
            left: 0;
            z-index: 999;
        }

        .map {
            position: absolute;
            width: 100%;
            height: 100%;
        }
    }
</style>