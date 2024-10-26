<script setup>
import AppLayout from "@/Layouts/AppLayout.vue";
import MapComponent from "@/Components/Items/Map.vue";

const props = defineProps({
    order: {
        type: Object,
        required: true,
    },
});
</script>

<template>
    <AppLayout title="Detalles de la Orden">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Detalles de la Orden #{{ order.id }}
            </h2>
        </template>

        <div class="py-12">
            <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
                <div class="bg-white shadow-md rounded-lg p-6">
                    <h3 class="text-lg font-semibold text-gray-700">
                        Información de la Orden
                    </h3>
                    <div class="mt-4">
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div class="border p-4 rounded-lg shadow-sm">
                                <p class="text-sm font-medium text-gray-500">
                                    Nombre del Cliente
                                </p>
                                <p class="text-lg text-gray-900">
                                    {{ order.client_name }}
                                </p>
                            </div>
                            <div class="border p-4 rounded-lg shadow-sm">
                                <p class="text-sm font-medium text-gray-500">
                                    Estado
                                </p>
                                <p class="text-lg text-gray-900">
                                    {{ order.status }}
                                </p>
                            </div>
                            <div class="border p-4 rounded-lg shadow-sm">
                                <p class="text-sm font-medium text-gray-500">
                                    Tipo de Envío
                                </p>
                                <p class="text-lg text-gray-900">
                                    {{ order.shipping_type }}
                                </p>
                            </div>
                            <div class="border p-4 rounded-lg shadow-sm">
                                <p class="text-sm font-medium text-gray-500">
                                    Monto Total
                                </p>
                                <p class="text-lg text-gray-900">
                                    ${{ order.total_amount.toFixed(2) }}
                                </p>
                            </div>
                            <div class="border p-4 rounded-lg shadow-sm">
                                <p class="text-sm font-medium text-gray-500">
                                    Dirección
                                </p>
                                <p class="text-lg text-gray-900">
                                    {{ order.address }}
                                </p>
                            </div>
                            <div class="border p-4 rounded-lg shadow-sm">
                                <p class="text-sm font-medium text-gray-500">
                                    Notas
                                </p>
                                <p class="text-lg text-gray-900">
                                    {{
                                        order.notes ? order.notes : "Sin notas"
                                    }}
                                </p>
                            </div>
                            <div class="border p-4 rounded-lg shadow-sm">
                                <p class="text-sm font-medium text-gray-500">
                                    Fecha de Creación
                                </p>
                                <p class="text-lg text-gray-900">
                                    {{
                                        new Date(
                                            order.created_at
                                        ).toLocaleString()
                                    }}
                                </p>
                            </div>
                            <div class="border p-4 rounded-lg shadow-sm">
                                <p class="text-sm font-medium text-gray-500">
                                    Última Actualización
                                </p>
                                <p class="text-lg text-gray-900">
                                    {{
                                        new Date(
                                            order.updated_at
                                        ).toLocaleString()
                                    }}
                                </p>
                            </div>
                        </div>
                    </div>
                    <MapComponent
                        class="mt-4"
                        :order="order"
                    />
                </div>
            </div>
        </div>
    </AppLayout>
</template>
