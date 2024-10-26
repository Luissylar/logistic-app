<template>
    <div>
        <div id="map" style="height: 500px"></div>
        <div v-if="duration" class="duration-info">
            Duración estimada: {{ duration }} minutos
        </div>
        <div v-if="weather" class="weather-info border p-4 rounded-lg shadow-sm bg-blue-100 text-blue-900">
            <p>Clima actual: <strong>{{ weather.description }}</strong></p>
            <p>Temperatura: <strong>{{ weather.temp }} °C</strong></p>
        </div>
    </div>
</template>

<script setup>
import { onMounted, defineProps, ref } from "vue";
import "leaflet/dist/leaflet.css";
import L from "leaflet";

function decodePolyline(encoded) {
    let coordinates = [];
    let index = 0,
        lat = 0,
        lng = 0;

    while (index < encoded.length) {
        let b,
            shift = 0,
            result = 0;

        do {
            b = encoded.charCodeAt(index++) - 63;
            result |= (b & 0x1f) << shift;
            shift += 5;
        } while (b >= 0x20);

        const dlat = (result >> 1) ^ -(result & 1);
        lat += dlat;

        shift = 0;
        result = 0;

        do {
            b = encoded.charCodeAt(index++) - 63;
            result |= (b & 0x1f) << shift;
            shift += 5;
        } while (b >= 0x20);

        const dlng = (result >> 1) ^ -(result & 1);
        lng += dlng;

        coordinates.push([lat / 1e5, lng / 1e5]);
    }

    return coordinates;
}

const props = defineProps({
    order: {
        type: Object,
        required: true,
    },
});

let map;
const duration = ref(null);
const weather = ref(null);

onMounted(() => {
    map = L.map("map").setView(
        [props.order.latitude, props.order.longitude],
        13
    );

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        maxZoom: 19,
        attribution: "© OpenStreetMap",
    }).addTo(map);

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

            map.setView([latitude, longitude], 13);

            L.marker([latitude, longitude])
                .addTo(map)
                .bindPopup("¡Estás aquí!")
                .openPopup();

            const clientLatitude = props.order.latitude;
            const clientLongitude = props.order.longitude;

            if (clientLatitude && clientLongitude) {
                L.marker([clientLatitude, clientLongitude])
                    .addTo(map)
                    .bindPopup(`Orden para: ${props.order.client_name}`)
                    .openPopup();

                drawRoute(
                    [latitude, longitude],
                    [clientLatitude, clientLongitude]
                );
                fetchWeather(latitude, longitude);
            } else {
                alert("Coordenadas del cliente no válidas.");
            }
        } catch (error) {
            alert("No se pudo obtener la ubicación: " + error.message);
        }
    };

    requestLocationPermission();

    const drawRoute = (start, end) => {
        const routeUrl = `https://router.project-osrm.org/route/v1/driving/${start[1]},${start[0]};${end[1]},${end[0]}?overview=full&geometries=polyline&steps=true&annotations=false`;

        fetch(routeUrl)
            .then((response) => {
                if (!response.ok) {
                    throw new Error("Error en la respuesta de la API");
                }
                return response.json();
            })
            .then((data) => {
                if (data.routes && data.routes.length > 0) {
                    const encodedRoute = data.routes[0].geometry;
                    const latLngs = decodePolyline(encodedRoute);
                    const routeLine = L.polyline(latLngs, {
                        color: "blue",
                        weight: 7,
                    }).addTo(map);

                    map.fitBounds(routeLine.getBounds());
                    const durationInSeconds = data.routes[0].duration;
                    duration.value = Math.round(durationInSeconds / 60);
                } else {
                    alert(
                        "No se pudo encontrar la ruta. Verifique las coordenadas."
                    );
                }
            })
            .catch((error) => {
                console.error("Error al obtener la ruta:", error);
                alert("Error al obtener la ruta: " + error.message);
            });
    };

    const fetchWeather = async (latitude, longitude) => {
        const url = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current_weather=true`;

        try {
            const response = await fetch(url);
            if (!response.ok) {
                throw new Error("Error al obtener el clima");
            }
            const data = await response.json();
            weather.value = {
                temp: data.current_weather.temperature,
                description: data.current_weather.weathercode, 
            };
        } catch (error) {
            console.error("Error al obtener el clima:", error);
        }
    };
});
</script>

<style scoped>
#map {
    height: 100vh;
}
.duration-info {
    margin-top: 10px;
    font-size: 16px;
    font-weight: bold;
}
.weather-info {
    margin-top: 10px;
    font-size: 16px;
    font-weight: bold;
}
</style>
