<script>
    import { ref, onMounted, onUnmounted, markRaw } from 'vue';
    import { Map, MapStyle, Marker, config } from '@maptiler/sdk';
    import '@maptiler/sdk/dist/maptiler-sdk.css';
    import axios from 'axios';

    export default {
        data() {
            return {
                mapContainer: null,
                map: null,
                form: {
                    title: '',
                    description: '',
                    image: null,
                    index: 0
                },
                selectedMarker: null,
                showModal: false,
                activeIndex: 0,
                locations: [] 
            };
        },

        methods: {
            initializeMap() {
                const mapContainer = this.$refs.mapContainer;

                if (!mapContainer) {
                    console.error('Map container not found');
                    return;
                }

                config.apiKey = 'cXh6kpqsIV9qrsbI2IKs';
                const initialState = { lng: 18.05688, lat: 42.35505, zoom: 5.2 };

                this.map = markRaw(new Map({
                    container: mapContainer,
                    style: MapStyle.STREETS,
                    center: [initialState.lng, initialState.lat],
                    zoom: initialState.zoom
                }));

                const storedCoordinates = this.getStoredCoordinates();
                this.locations = storedCoordinates;

                storedCoordinates.forEach(({ lat, lon, title, description, image, index }) => {
                    this.addMarker(lat, lon, title, description, image, index);
                });
            },

            async saveForm() {
                const address = this.form.title;
                const description = this.form.description;
                const image = this.form.image;
                const index = this.form.index;

                if (!address) {
                    console.log('Inserisci un indirizzo');
                    return;
                }

                try {
                    const response = await axios.get(`https://api.tomtom.com/search/2/geocode/${encodeURIComponent(address)}.json?key=SvIJjQs1GXAO5L5EXzIoRElBWyGEsSX2`);
                    if (response.data.results && response.data.results.length > 0) {
                        const { lat, lon } = response.data.results[0].position;
                        this.addMarker(lat, lon, address, description, image, index);
                        this.storeCoordinates(lat, lon, address, description, image, index);

                        this.locations.push({ lat, lon, title: address, description, image, index });
                    } else {
                        console.log('Indirizzo non trovato');
                    }
                } catch (error) {
                    console.error('Richiesta fallita', error);
                }
            },

            addMarker(lat, lon, title, description, image, index) {
                if (this.map) {
                    const marker = new Marker({ color: '#9c248e' })
                        .setLngLat([lon, lat])
                        .addTo(this.map);

                    marker.getElement().addEventListener('click', () => {
                        this.selectedMarker = { marker, title, description, image, lat, lon, index };
                        this.activeIndex = index;
                        this.showModal = true;
                    });
                }
            },

            storeCoordinates(lat, lon, title, description, image, index) {
                const coordinates = this.getStoredCoordinates() || [];
                coordinates.push({ lat, lon, title, description, image, index });
                localStorage.setItem('coordinates', JSON.stringify(coordinates));
            },

            getStoredCoordinates() {
                const serializedCoordinates = localStorage.getItem('coordinates');
                return serializedCoordinates ? JSON.parse(serializedCoordinates) : [];
            },

            closeModal() {
                this.showModal = false;
                this.selectedMarker = null;
            },

            deleteMarker() {
                if (this.selectedMarker) {
                    const { marker, lat, lon, title, description, image, index } = this.selectedMarker;

                    marker.remove();

                    let coordinates = this.getStoredCoordinates();
                    coordinates = coordinates.filter(coord => coord.lat !== lat || coord.lon !== lon || coord.title !== title || coord.description !== description || coord.image !== image || coord.index !== index);
                    localStorage.setItem('coordinates', JSON.stringify(coordinates));

                    this.locations = coordinates;

                    this.closeModal();
                }
            },

            updateDescription() {
                if (this.selectedMarker) {
                    const { lat, lon, title, description, image, index } = this.selectedMarker;
                    let coordinates = this.getStoredCoordinates();
                    coordinates = coordinates.map(coord => {
                        if (coord.lat === lat && coord.lon === lon && coord.title === title) {
                            return { ...coord, description, image, index: this.activeIndex };
                        }
                        return coord;
                    });

                    localStorage.setItem('coordinates', JSON.stringify(coordinates));

                    this.locations = coordinates;

                    this.closeModal();
                }
            },

            saveIndex(index) {
                this.activeIndex = index + 1;
                if (this.selectedMarker) {
                    this.selectedMarker.index = this.activeIndex;
                } else {
                    this.form.index = this.activeIndex;
                }
            },

            handleImageUpload(event) {
                const file = event.target.files[0];
                if (file.size > 800 * 1024) {
                    alert('La dimensione dell\'immagine non deve superare 800 KB');
                    return;
                }
                const reader = new FileReader();
                reader.onload = (e) => {
                    this.form.image = e.target.result;
                };
                if (file) {
                    reader.readAsDataURL(file);
                }
            },

            handleMarkerImageUpload(event) {
                const file = event.target.files[0];
                if (file.size > 800 * 1024) {
                    alert('La dimensione dell\'immagine non deve superare 800 KB');
                    return;
                }
                const reader = new FileReader();
                reader.onload = (e) => {
                    this.selectedMarker.image = e.target.result;
                };
                if (file) {
                    reader.readAsDataURL(file);
                }
            }
        },

        mounted() {
            this.initializeMap();
        },

        unmounted() {
            if (this.map) {
                this.map.remove();
            }
        }
    };
</script>

<template>
    <div class="map-wrap">
        <!-- Add Location Button -->
        <button type="button" class="btn btn-default btn-circle btn-lg ms-btn" data-bs-toggle="modal"
            data-bs-target="#addLocationModal">
            <i class="fa fa-plus"></i>
        </button>

        <!-- Location Cards -->
        <div class="container mt-4 position-absolute z-3 ms-card-container">
            <div class="row">
                <div class="col mb-4 ms-card" v-for="(location, index) in locations" :key="index">
                    <div class="card">
                        <img :src="location.image" class="card-img-top img-fluid object-fit-cover ms-img" alt="Image" v-if="location.image">
                        <div class="card-body">
                            <h5 class="card-title">{{ location.title }}</h5>
                            <p class="card-text">{{ location.description }}</p>
                            <div class="vote-container">
                                <i class="fa-solid fa-star" v-for="starIndex in 5" :key="starIndex"
                                    :class="{ 'active-star': starIndex <= location.index }"></i>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

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
                        <button type="submit" class="btn btn-primary" @click.prevent="saveForm"
                            data-bs-dismiss="modal">Salva</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Map Container -->
        <div ref="mapContainer" class="map-container"></div>

        <!-- Marker Details Modal -->
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
                                @click="saveIndex(index)" :class="{ 'active-star': index < selectedMarker.index }"></i>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" @click="closeModal">Chiudi</button>
                        <button type="button" class="btn btn-primary" @click="updateDescription">Salva
                            Modifiche</button>
                        <button type="button" class="btn btn-danger" @click="deleteMarker">Elimina</button>
                    </div>
                </div>
            </div>
        </div>
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
        height: calc(100vh - 159px);
    }

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

    .modal-backdrop.show {
        opacity: 0.5;
    }

    .map-container {
        width: 100%;
        height: 100%;
    }

    .vote-container {
        display: flex;
        gap: 5px;
    }

    .fa-star {
        font-size: 1.5em;
        cursor: pointer;
        color: gray;
    }

    .active-star {
        color: orange;
    }

    .ms-card-container {
        bottom: 20px;

        .ms-card {
            display: flex;
            flex-direction: column;
            align-items: stretch;
            height: 100%; 

            .card {
                height: 100%;
                display: flex;
                flex-direction: column;
                justify-content: space-between; 
                background: white;

                .ms-img {
                    height: 150px;
                    object-fit: cover;
                }

                .card-body {
                    flex-grow: 1;
                    display: flex;
                    flex-direction: column;
                    justify-content: space-between;
                }
            }
        }
    }
</style>
