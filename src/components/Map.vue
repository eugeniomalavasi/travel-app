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

    const selectedMarker = ref(null); // Stato per il marker selezionato
    const showModal = ref(false); // Stato per il modale

    onMounted(() => {
        config.apiKey = 'cXh6kpqsIV9qrsbI2IKs';

        const initialState = { lng: 18.05688, lat: 42.35505, zoom: 5.2 };

        map.value = markRaw(new Map({
            container: mapContainer.value,
            style: MapStyle.STREETS,
            center: [initialState.lng, initialState.lat],
            zoom: initialState.zoom
        }));

        // Aggiungi i marker memorizzati
        const storedCoordinates = getStoredCoordinates();
        storedCoordinates.forEach(({ lat, lon, title, description }) => {
            addMarker(lat, lon, title, description);
        });
    });

    onUnmounted(() => {
        map.value?.remove();
    });

    const save_form = async () => {
        const address = form.value.title;
        const description = form.value.description;
        if (!address) {
            console.log('Inserisci un indirizzo');
            return;
        }

        try {
            const response = await axios.get(`https://api.tomtom.com/search/2/geocode/${encodeURIComponent(address)}.json?key=SvIJjQs1GXAO5L5EXzIoRElBWyGEsSX2`);
            if (response.data.results && response.data.results.length > 0) {
                const { lat, lon } = response.data.results[0].position;
                addMarker(lat, lon, address, description);
                storeCoordinates(lat, lon, address, description);
            } else {
                console.log('Indirizzo non trovato');
            }
        } catch (error) {
            console.error('Richiesta fallita', error);
        }
    };

    const addMarker = (lat, lon, title, description) => {
        if (map.value) {
            const marker = new Marker({ color: "blue" })
                .setLngLat([lon, lat])
                .addTo(map.value);

            // Aggiungi evento click al marker
            marker.getElement().addEventListener('click', () => {
                selectedMarker.value = { marker, title, description, lat, lon };
                showModal.value = true;
            });
        }
    };

    const storeCoordinates = (lat, lon, title, description) => {
        const coordinates = getStoredCoordinates() || [];
        coordinates.push({ lat, lon, title, description });
        localStorage.setItem('coordinates', JSON.stringify(coordinates));
    };

    const getStoredCoordinates = () => {
        const serializedCoordinates = localStorage.getItem('coordinates');
        return serializedCoordinates ? JSON.parse(serializedCoordinates) : [];
    };

    const closeModal = () => {
        showModal.value = false;
        selectedMarker.value = null;
    };

    const deleteMarker = () => {
        if (selectedMarker.value) {
            const { marker, lat, lon, title, description } = selectedMarker.value;

            marker.remove();

            let coordinates = getStoredCoordinates();
            coordinates = coordinates.filter(coord => coord.lat !== lat || coord.lon !== lon || coord.title !== title || coord.description !== description);
            localStorage.setItem('coordinates', JSON.stringify(coordinates));

            closeModal();
        }
    };
</script>

<template>
    <div class="map-wrap">
        <!-- Button trigger modal -->
        <button type="button" class="btn btn-primary ms-btn" data-bs-toggle="modal" data-bs-target="#addLocationModal">
            Aggiungi Località
        </button>

        <!-- Add Location Modal -->
        <div class="modal fade" id="addLocationModal" tabindex="-1" aria-labelledby="addLocationModalLabel"
            aria-hidden="true">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h1 class="modal-title fs-5" id="addLocationModalLabel">Informazioni località</h1>
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
                        <button type="submit" class="btn btn-primary" @click.prevent="save_form"
                            data-bs-dismiss="modal">Salva</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Marker Info Modal -->
        <div class="modal" :class="{ show: showModal }" tabindex="-1" style="display: block;" v-if="showModal">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h1 class="modal-title fs-5">Informazioni Marker</h1>
                        <button type="button" class="btn-close" @click="closeModal"></button>
                    </div>
                    <div class="modal-body">
                        <p><strong>Indirizzo:</strong> {{ selectedMarker?.title }}</p>
                        <p><strong>Descrizione:</strong> {{ selectedMarker?.description }}</p>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" @click="closeModal">Chiudi</button>
                        <button type="button" class="btn btn-danger" @click="deleteMarker">Elimina</button>
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

    .modal.show {
        display: block;
        background: rgba(0, 0, 0, 0.5);
    }
</style>