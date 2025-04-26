<template>
  <div class="principal-reservas">
    <FormularioReserva 
      :reservas="reservas" 
      @guardar-reserva="agregarReserva" 
    />
    <TimelineReservas
      :reservas="reservas"
    />
    <TablaReservas
      :reservas="reservas"
      @eliminar-reserva="eliminarReserva"
    />
  </div>
</template>

<script setup>
/* global moment */
import { ref } from 'vue';
import FormularioReserva from "./FormularioReserva.vue";
import TimelineReservas from "./TimelineReservas.vue";
import TablaReservas from "./TablaReservas.vue";


const nextId = ref(2);
const reservas = ref([
  {
    id: 1,
    content: "Acuña López, Juliana",
    nombre: "Acuña López, Juliana",
    fecha: moment().utcOffset(0).format("YYYY-MM-DD"), // UTC+0
    horaInicio: "11:00",
    horaFin: "12:30",
    start: moment().utcOffset(0).set({ hour: 11, minute: 0 }).format("YYYY-MM-DD HH:mm"),
    end: moment().utcOffset(0).set({ hour: 12, minute: 30 }).format("YYYY-MM-DD HH:mm")
  }
]);

function agregarReserva(nuevaReserva) {
  // Asignamos un ID único a la reserva
  const reservaConId = {
    ...nuevaReserva,
    id: nextId.value++
  };
  
  // Añadimos la reserva al array
  reservas.value.push(reservaConId);
}

function eliminarReserva(id) {
  reservas.value = reservas.value.filter(r => r.id !== id);
}
</script>

<style scoped>
.principal-reservas {
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
}
</style>