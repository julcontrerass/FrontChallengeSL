<template>
  <div class="table-box">
    <table class="table-reservations">
      <thead>
        <tr>
          <th>Nombre</th>
          <th>Horario <i class="sort-icon">↑</i></th>
          <th>Acción</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in reservations" :key="item.id">
          <td>{{ item.content }}</td>
          <td>
            de {{ formatSchedule(item.start) }} a {{ formatSchedule(item.end) }}
          </td>
          <td>
            <button class="delete" @click="requestDelete(item.id)">
              <i class="fas fa-trash-alt"></i>
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
/* global moment*/
export default {
  name: "TableReservations",
  props: {
    reservations: {
      type: Array,
      required: true,
    },
  },
  emits: ["delete-reservation"],
  setup(props, { emit }) {

    function formatSchedule(date) {
      return moment(date).locale("es").format("HH:mm");
    }

    function requestDelete(id) {
      if (confirm("¿Estás seguro de eliminar esta reserva?")) {
        emit("delete-reservation", id);
      }
    }

    return {
      formatSchedule,
      requestDelete,
    };
  },
};
</script>

<style scoped>
.table-box {
  width: 100%;
  max-width: 900px;
  margin-top: 30px;
}

.table-reservations {
  width: 100%;
  border-collapse: collapse;
  background: white;
  border-radius: 6px;
  overflow: hidden;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.table-reservations th,
.table-reservations td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #eee;
}

.table-reservations th {
  background-color: #f8f8f8;
  font-weight: 500;
  color: #444;
  font-size: 14px;
}

.table-reservations tbody tr:hover {
  background-color: #f9f9f9;
}

.sort-icon {
  font-size: 19px;
  margin-left: 4px;
  color: #999;
}

.delete {
  background: none;
  border: none;
  cursor: pointer;
  color: #bbb;
  padding: 5px;
  font-size: 14px;
  transition: color 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.delete:hover {
  color: #e74c3c;
}
</style>