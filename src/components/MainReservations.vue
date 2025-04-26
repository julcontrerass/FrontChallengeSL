<template>
  <div class="main-reservations">
    <FormReservation 
      :reservations="reservations" 
      @save-reservation="addReservation" 
    />
    <TimelineReservations
      :reservations="reservations"
    />
    <TableReservations
      :reservations="reservations"
      @delete-reservation="deleteReservation"
    />
  </div>
</template>

<script setup>
/* global moment */
import { ref } from 'vue';
import FormReservation from "./FormReservation.vue";
import TimelineReservations from "./TimelineReservations.vue";
import TableReservations from "./TableReservations.vue";

const nextId = ref(2);
const reservations = ref([
  {
    id: 1,
    content: "Acuña López, Juliana",
    name: "Acuña López, Juliana",
    date: moment().utcOffset(0).format("YYYY-MM-DD"),
    startTime: "11:00",
    endTime: "12:30",
    start: moment().utcOffset(0).set({ hour: 11, minute: 0 }).format("YYYY-MM-DD HH:mm"),
    end: moment().utcOffset(0).set({ hour: 12, minute: 30 }).format("YYYY-MM-DD HH:mm")
  }
]);

function addReservation(newReservation) {
  // Asignar ID a la nueva reserva
  const reservationWithId = {
    ...newReservation,
    id: nextId.value++
  };
  
  // Agregar la reserva al array
  reservations.value.push(reservationWithId);
}

function deleteReservation(id) {
  reservations.value = reservations.value.filter(r => r.id !== id);
}
</script>

<style scoped>
.main-reservations {
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
}
</style>