<template>
    <AppLayout title="Dashboard">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Crear Orden
            </h2>
        </template>
        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white overflow-hidden shadow-xl sm:rounded-lg p-6">
                    <!-- Campo de búsqueda de dirección -->
                    <div class="mb-4">
                        <label for="address" class="block text-sm font-medium text-gray-700">Buscar Dirección</label>
                        <div class="flex mt-1">
                            <input
                                id="address"
                                type="text"
                                v-model="form.address"
                                @keydown.enter.prevent="searchAddress"
                                placeholder="Ingrese dirección"
                                class="flex-1 border border-gray-300 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
                            />
                            <button @click="searchAddress" class="ml-2 bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-500">
                                Buscar
                            </button>
                        </div>
                    </div>

                    <!-- Mapa -->
                    <div id="map" class="mb-4" style="height: 500px"></div>

                    <!-- Formulario de orden -->
                    <form @submit.prevent="submitOrder" class="space-y-4">
                        <div>
                            <label for="clientName" class="block text-sm font-medium text-gray-700">Nombre del Cliente</label>
                            <input
                                id="clientName"
                                type="text"
                                v-model="form.client_name"
                                placeholder="Nombre del Cliente"
                                required
                                class="mt-1 border border-gray-300 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-blue-500 w-full"
                            />
                            <InputError class="mt-2" :message="form.errors.client_name" />
                        </div>

                        <div>
                            <label for="status" class="block text-sm font-medium text-gray-700">Estado</label>
                            <select
                                id="status"
                                v-model="form.status"
                                required
                                class="mt-1 border border-gray-300 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-blue-500 w-full"
                            >
                                <option value="pendiente">Pendiente</option>
                                <option value="en proceso">En Proceso</option>
                                <option value="completado">Completado</option>
                                <option value="cancelado">Cancelado</option>
                            </select>
                            <InputError class="mt-2" :message="form.errors.status" />
                        </div>

                        <div>
                            <label for="shippingType" class="block text-sm font-medium text-gray-700">Tipo de Envío</label>
                            <select
                                id="shippingType"
                                v-model="form.shipping_type"
                                required
                                class="mt-1 border border-gray-300 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-blue-500 w-full"
                            >
                                <option value="aereo">Aéreo</option>
                                <option value="terrestre">Terrestre</option>
                            </select>
                            <InputError class="mt-2" :message="form.errors.shipping_type" />
                        </div>

                        <div>
                            <label for="totalAmount" class="block text-sm font-medium text-gray-700">Monto Total</label>
                            <input
                                id="totalAmount"
                                type="number"
                                v-model="form.total_amount"
                                placeholder="Monto Total"
                                required
                                class="mt-1 border border-gray-300 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-blue-500 w-full"
                            />
                            <InputError class="mt-2" :message="form.errors.total_amount" />
                        </div>

                        <input type="hidden" v-model="form.latitude" name="latitude" />
                        <input type="hidden" v-model="form.longitude" name="longitude" />

                        <div>
                            <label for="orderAddress" class="block text-sm font-medium text-gray-700">Dirección</label>
                            <input
                                id="orderAddress"
                                type="text"
                                v-model="form.address"
                                name="address"
                                placeholder="Dirección"
                                required
                                class="mt-1 border border-gray-300 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-blue-500 w-full"
                            />
                            <InputError class="mt-2" :message="form.errors.address" />
                        </div>

                        <div>
                            <label for="notes" class="block text-sm font-medium text-gray-700">Notas (opcional)</label>
                            <textarea
                                id="notes"
                                v-model="form.notes"
                                placeholder="Notas (opcional)"
                                class="mt-1 border border-gray-300 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-blue-500 w-full"
                                rows="3"
                            ></textarea>
                            <InputError class="mt-2" :message="form.errors.notes" />
                        </div>

                        <button
                            type="submit"
                            class="w-full bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-500"
                        >
                            Crear Orden
                        </button>
                    </form>
                </div>
            </div>
        </div>
    </AppLayout>
</template>

<script setup>
import AppLayout from "@/Layouts/AppLayout.vue";
import { ref, onMounted } from "vue";
import { useForm } from "@inertiajs/vue3";
import InputError from '@/Components/InputError.vue';
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import axios from "axios";

const map = ref(null);
const marker = ref(null);
const location = ref({ lat: null, lng: null });

// Utiliza useForm para crear el formulario
const form = useForm({
    address: "",
    client_name: "",
    status: "pendiente",
    shipping_type: "terrestre",
    total_amount: null,
    notes: "",
    latitude: null, // Asegúrate de incluir estos campos en el formulario
    longitude: null,
});

onMounted(() => {
    map.value = L.map("map").setView([51.505, -0.09], 13);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        maxZoom: 19,
        attribution: "© OpenStreetMap",
    }).addTo(map.value);

    // Agregar marcador en el mapa al hacer clic
    map.value.on("click", (e) => {
        setMarker(e.latlng);
    });
});

// Función para buscar una dirección y centrar el mapa
const searchAddress = async () => {
    try {
        const response = await axios.get(
            `https://nominatim.openstreetmap.org/search?format=json&q=${form.address}`
        );
        if (response.data.length) {
            const { lat, lon } = response.data[0];
            setMarker([lat, lon]);
            map.value.setView([lat, lon], 13);
        } else {
            alert("Dirección no encontrada");
        }
    } catch (error) {
        alert("Error al buscar la dirección");
    }
};

// Función para establecer un marcador y actualizar la ubicación seleccionada
const setMarker = (latlng) => {
    if (marker.value) {
        marker.value.setLatLng(latlng);
    } else {
        marker.value = L.marker(latlng).addTo(map.value);
    }
    marker.value.bindPopup("Ubicación seleccionada").openPopup();
    // Asignar latitud y longitud al formulario
    form.latitude = latlng.lat;
    form.longitude = latlng.lng;
    location.value = { lat: latlng.lat, lng: latlng.lng };
};

// Función para enviar la orden y guardar la ubicación
const submitOrder = () => {
    // Enviar el formulario a la ruta de almacenamiento usando Inertia
    form.post(route("orders.store"), {
        onSuccess: () => {
            alert("Orden creada con éxito");
        },
        onError: (errors) => {
            console.error(errors);
            // Aquí podrías manejar otros tipos de errores si es necesario
        },
    });
};
</script>

<style scoped>
#map {
    height: 500px; /* Ajusta la altura según tus necesidades */
}
</style>
