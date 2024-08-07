<script setup>
    import { ref, onMounted, onUnmounted, markRaw } from 'vue';
    import { Map, MapStyle, Marker, config } from '@maptiler/sdk';
    import '@maptiler/sdk/dist/maptiler-sdk.css';
    import axios from 'axios';

    const mapContainer = ref(null);
    const map = ref(null);

    const form = ref({
        title: '',
        description: '',
        image: null,
        index: 0
    });

    const selectedMarker = ref(null);
    const showModal = ref(false);
    const activeIndex = ref(0);

    onMounted(() => {
        config.apiKey = 'cXh6kpqsIV9qrsbI2IKs';

        const initialState = { lng: 18.05688, lat: 42.35505, zoom: 5.2 };

        map.value = markRaw(new Map({
            container: mapContainer.value,
            style: MapStyle.STREETS,
            center: [initialState.lng, initialState.lat],
            zoom: initialState.zoom
        }));

        const storedCoordinates = getStoredCoordinates();
        storedCoordinates.forEach(({ lat, lon, title, description, image, index }) => {
            addMarker(lat, lon, title, description, image, index);
        });
    });

    onUnmounted(() => {
        map.value?.remove();
    });

    const save_form = async () => {
        const address = form.value.title;
        const description = form.value.description;
        const image = form.value.image;
        const index = form.value.index;

        if (!address) {
            console.log('Inserisci un indirizzo');
            return;
        }

        try {
            const response = await axios.get(`https://api.tomtom.com/search/2/geocode/${encodeURIComponent(address)}.json?key=SvIJjQs1GXAO5L5EXzIoRElBWyGEsSX2`);
            if (response.data.results && response.data.results.length > 0) {
                const { lat, lon } = response.data.results[0].position;
                addMarker(lat, lon, address, description, image, index);
                storeCoordinates(lat, lon, address, description, image, index);
            } else {
                console.log('Indirizzo non trovato');
            }
        } catch (error) {
            console.error('Richiesta fallita', error);
        }
    };

    const addMarker = (lat, lon, title, description, image, index) => {
        if (map.value) {
            const marker = new Marker({ color: "#9c248e" })
                .setLngLat([lon, lat])
                .addTo(map.value);

            marker.getElement().addEventListener('click', () => {
                selectedMarker.value = { marker, title, description, image, lat, lon, index };
                activeIndex.value = index; // Set the active index
                showModal.value = true;
            });
        }
    };

    const storeCoordinates = (lat, lon, title, description, image, index) => {
        const coordinates = getStoredCoordinates() || [];
        coordinates.push({ lat, lon, title, description, image, index });
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
            const { marker, lat, lon, title, description, image, index } = selectedMarker.value;

            marker.remove();

            let coordinates = getStoredCoordinates();
            coordinates = coordinates.filter(coord => coord.lat !== lat || coord.lon !== lon || coord.title !== title || coord.description !== description || coord.image !== image || coord.index !== index);
            localStorage.setItem('coordinates', JSON.stringify(coordinates));

            closeModal();
        }
    };

    const updateDescription = () => {
        if (selectedMarker.value) {
            const { lat, lon, title, description, image, index } = selectedMarker.value;
            let coordinates = getStoredCoordinates();
            coordinates = coordinates.map(coord => {
                if (coord.lat === lat && coord.lon === lon && coord.title === title) {
                    return { ...coord, description, image, index: activeIndex.value };
                }
                return coord;
            });

            localStorage.setItem('coordinates', JSON.stringify(coordinates));

            closeModal();
        }
    }

    const saveIndex = (index) => {
        activeIndex.value = index + 1;
        if (selectedMarker.value) {
            selectedMarker.value.index = activeIndex.value;
        } else {
            form.value.index = activeIndex.value;
        }
    }

    const handleImageUpload = (event) => {
        const file = event.target.files[0];
        const reader = new FileReader();
        reader.onload = (e) => {
            form.value.image = e.target.result;
        };
        if (file) {
            reader.readAsDataURL(file);
        }
    }

    const handleMarkerImageUpload = (event) => {
        const file = event.target.files[0];
        const reader = new FileReader();
        reader.onload = (e) => {
            selectedMarker.value.image = e.target.result;
        };
        if (file) {
            reader.readAsDataURL(file);
        }
    }
</script>

<template>
    <div class="map-wrap">
        <button type="button" class="btn btn-default btn-circle btn-lg ms-btn" data-bs-toggle="modal"
            data-bs-target="#addLocationModal">
            <i class="fa fa-plus"></i>
        </button>
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
                            <label for="image">Immagine</label>
                            <input type="file" @change="handleImageUpload" class="form-control" id="image">
                            <div class="vote-container mt-2">
                                <i class="fa-solid fa-star" v-for="(star, index) in 5" :key="index"
                                    @click="saveIndex(index)" :class="{ 'active-star': index < form.index }"></i>
                            </div>
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
                        <h1 class="modal-title fs-5">Dettagli</h1>
                        <button type="button" class="btn-close" @click="closeModal"></button>
                    </div>
                    <div class="modal-body">
                        <p><strong>Indirizzo:</strong> {{ selectedMarker?.title }}</p>
                        <p><strong>Descrizione:</strong></p>
                        <textarea v-model="selectedMarker.description" class="form-control"></textarea>
                        <label for="markerImage">Immagine</label>
                        <input type="file" @change="handleMarkerImageUpload" class="form-control" id="markerImage">
                        <div v-if="selectedMarker?.image" class="mt-2">
                            <img :src="selectedMarker.image" alt="Marker Image" class="img-fluid">
                        </div>
                        <div class="vote-container mt-2">
                            <i class="fa-solid fa-star" v-for="(star, index) in 5" :key="index"
                                @click="saveIndex(index)" :class="{ 'active-star': index < activeIndex }"></i>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" @click="closeModal">Chiudi</button>
                        <button type="button" class="btn btn-danger" @click="deleteMarker">Elimina</button>
                        <button type="button" class="btn btn-primary" @click="updateDescription">Salva
                            modifiche</button>
                    </div>
                </div>
            </div>
        </div>

        <div class="map" ref="mapContainer"></div>
    </div>
</template>

<style scoped lang="scss">
    @import url('https://fonts.googleapis.com/css2?family=Varela+Round&display=swap');

    * {
        font-family: "Varela Round", sans-serif;
        font-weight: 400;
        font-style: normal;
    }

    .map-wrap {
        position: relative;
        width: 100%;
        height: calc(100vh - 56px);

        .ms-btn {
            position: absolute;
            right: 30px;
            bottom: 80px;
            z-index: 999;
            width: 70px;
            height: 70px;
            padding: 10px 16px;
            border-radius: 35px;
            font-size: 24px;
            line-height: 1.33;
            background-color: #9c248e;
            color: white;
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

    .vote-container {
        display: flex;
        gap: 0.5rem;
    }

    .fa-star {
        cursor: pointer;
        font-size: 1.5rem;
        color: grey;
    }

    .fa-star.active-star {
        color: orange;
    }
</style>