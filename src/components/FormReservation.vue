<template>
  <div class="form-box">
    <h2 class="form-title">RESERVAR</h2>
    <div class="form-fields">
      <div class="field">
        <label>Nombre</label>
        <input v-model="newReservation.name" type="text" placeholder="Nombre" />
      </div>
      <div class="field">
        <label>Fecha</label>
        <input v-model="newReservation.date" type="date" />
      </div>
      <div class="field">
        <label>Desde</label>
        <input v-model="newReservation.startTime" type="time" />
      </div>
      <div class="field">
        <label>Hasta</label>
        <input v-model="newReservation.endTime" type="time" />
      </div>
      <button @click="submitForm">Guardar</button>
    </div>
  </div>
</template>

<script>
/* global moment */
import { reactive } from 'vue';

export default {
  name: 'FormReservation',
  props: {
    reservations: {
      type: Array,
      required: true
    }
  },
  emits: ['save-reservation'],
  setup(props, { emit }) {
    const newReservation = reactive({
      name: '',
      date: new Date().toISOString().slice(0, 10),
      startTime: '',
      endTime: ''
    });

    function submitForm() {
      const { name, date, startTime, endTime } = newReservation;
      
      // Validacion de campos
      if (!name || !date || !startTime || !endTime) {
        return alert("Completa todos los campos.");
      }

      if (!moment) {
        alert("Error: La biblioteca moment.js no está disponible");
        return;
      }

      const start = moment(`${date} ${startTime}`);
      const end = moment(`${date} ${endTime}`);
      
      // Validacion de horarios
      if (!start.isBefore(end)) {
        return alert('"Desde" debe ser menor que "Hasta".');
      }

      // Verificar conflictos con otras reservas
      const hasOverlap = props.reservations.some(r => {
        const reservationStart = r.start ? moment(r.start) : moment(`${r.date} ${r.startTime}`);
        const reservationEnd = r.end ? moment(r.end) : moment(`${r.date} ${r.endTime}`);
        return reservationStart.isBefore(end) && reservationEnd.isAfter(start);
      });
      
      if (hasOverlap) {
        return alert("Ya hay una reserva en ese horario.");
      }

      // Crear objeto de reserva y emitirlo
      emit('save-reservation', {
        name: newReservation.name,
        content: newReservation.name, 
        date: newReservation.date,
        startTime: newReservation.startTime,
        endTime: newReservation.endTime,
        start: `${newReservation.date} ${newReservation.startTime}`, 
        end: `${newReservation.date} ${newReservation.endTime}`
      });
      
      // Limpiar formulario
      newReservation.name = '';
      newReservation.date = new Date().toISOString().slice(0, 10);
      newReservation.startTime = '';
      newReservation.endTime = '';
    }

    return {
      newReservation,
      submitForm
    };
  }
};
</script>

<style scoped>
.form-box {
  background: white;
  padding: 25px 20px;
  border-radius: 8px;
  margin-bottom: 30px;
  width: 100%;
  max-width: 900px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.form-title {
  text-align: left;
  margin-bottom: 20px;
  font-size: 20px;
  font-weight: bold;
  color: #006d5b;
  text-transform: uppercase;
}

.form-fields {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 10px 20px;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 0;
}

.field label {
  font-size: 14px;
  color: #555;
  font-weight: 500;
}

.form-fields input {
  height: 36px;
  padding: 0 10px;
  font-size: 14px;
  border-radius: 4px;
  border: 1px solid #ddd;
  min-width: 120px;
}

.field:nth-child(1) {
  flex: 1;
  min-width: 180px;
}

.field:nth-child(2) input {
  width: 140px;
}

.field:nth-child(3) input,
.field:nth-child(4) input {
  width: 80px;
}

.form-fields button {
  height: 36px;
  background-color: #00b894;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 0 20px;
  font-size: 14px;
  cursor: pointer;
  transition: background 0.2s;
  margin-left: auto;
  align-self: flex-end;
}

.form-fields button:hover {
  background-color: #009874;
}
</style>