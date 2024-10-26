<template>
    <div id="map" style="height: 500px"></div>
</template>

<script setup>
import { onMounted, defineProps } from "vue";
import "leaflet/dist/leaflet.css";
import L from "leaflet";

const props = defineProps({
    order: {
        type: Object,
        required: true,
    },
});

// Declare map here
let map; 

onMounted(() => {
    // Initialize the map
    map = L.map("map").setView([props.order.latitude, props.order.longitude], 13);

    // Add tile layer
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        maxZoom: 19,
        attribution: "© OpenStreetMap",
    }).addTo(map);

    // Request location permission
    const requestLocationPermission = async () => {
        try {
            const position = await new Promise((resolve, reject) => {
                if (navigator.geolocation) {
                    navigator.geolocation.getCurrentPosition(resolve, reject);
                } else {
                    reject(new Error("Geolocalización no soportada"));
                }
            });

            const { latitude, longitude } = position.coords;

            // Center the map at the user's location
            map.setView([latitude, longitude], 13);

            // Add a marker for the user's location
            L.marker([latitude, longitude])
                .addTo(map)
                .bindPopup("¡Estás aquí!")
                .openPopup();

            // Add a marker for the client's location
            L.marker([props.order.latitude, props.order.longitude])
                .addTo(map)
                .bindPopup(`Orden para: ${props.order.client_name}`)
                .openPopup();

            // Draw the route from the user's location to the client's location
            drawRoute([latitude, longitude], [props.order.latitude, props.order.longitude]);
        } catch (error) {
            alert("No se pudo obtener la ubicación: " + error.message);
        }
    };

    requestLocationPermission();

    // Define drawRoute function
    const drawRoute = (start, end) => {
        // Ensure 'map' is defined before using it
        if (!map) {
            console.error("El mapa no está definido.");
            return;
        }

        const routeUrl = `https://router.project-osrm.org/route/v1/driving/${start[1]},${start[0]};${end[1]},${end[0]}?overview=full`;

        fetch(routeUrl)
            .then((response) => {
                if (!response.ok) {
                    throw new Error("Error en la respuesta de la API");
                }
                return response.json();
            })
            .then((data) => {
                // Check if data.routes exists and has routes
                if (data.routes && data.routes.length > 0) {
                    const route = data.routes[0].geometry.coordinates;
                    const latLngs = route.map((coord) => [coord[1], coord[0]]);

                    // Draw the route on the map
                    const routeLine = L.polyline(latLngs, {
                        color: "blue",
                        weight: 4,
                    }).addTo(map);
                    map.fitBounds(routeLine.getBounds()); // Adjust the map to show the route
                } else {
                    alert("No se pudo encontrar la ruta. Verifique las coordenadas.");
                }
            })
            .catch((error) => {
                console.error("Error al obtener la ruta:", error);
                alert("Error al obtener la ruta: " + error.message);
            });
    };
});
</script>

<style scoped>
#map {
    height: 100vh;
}
</style>
