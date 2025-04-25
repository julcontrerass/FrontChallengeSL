<template>
  <div class="tabla-box">
    <table class="tabla-reservas">
      <thead>
        <tr>
          <th>Nombre</th>
          <th>Horario <i class="ordenar-icon">↑</i></th>
          <th>Acción</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in reservas" :key="item.id">
          <td>{{ item.content }}</td>
          <td>
            de {{ formatoHorario(item.start) }} a {{ formatoHorario(item.end) }}
          </td>
          <td>
            <button class="eliminar" @click="solicitarEliminar(item.id)">
              <i class="fas fa-trash-alt"></i>
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { onMounted } from "vue";
import moment from "../assets/vis/moment.js";

export default {
  name: "TablaReservas",
  props: {
    reservas: {
      type: Array,
      required: true,
    },
  },
  emits: ["eliminar-reserva"],
  setup(props, { emit }) {
    onMounted(() => {
      if (!document.getElementById("fontawesome-css")) {
        const link = document.createElement("link");
        link.id = "fontawesome-css";
        link.rel = "stylesheet";
        link.href =
          "https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css";
        document.head.appendChild(link);
      }
    });

    function formatoHorario(date) {
      return moment(date).locale("es").format("HH:mm");
    }

    function solicitarEliminar(id) {
      if (confirm("¿Estás seguro de eliminar esta reserva?")) {
        emit("eliminar-reserva", id);
      }
    }

    return {
      formatoHorario,
      solicitarEliminar,
    };
  },
};
</script>

<style scoped>
.tabla-box {
  width: 100%;
  max-width: 900px;
  margin-top: 30px;
}

.tabla-reservas {
  width: 100%;
  border-collapse: collapse;
  background: white;
  border-radius: 6px;
  overflow: hidden;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.tabla-reservas th,
.tabla-reservas td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #eee;
}

.tabla-reservas th {
  background-color: #f8f8f8;
  font-weight: 500;
  color: #444;
  font-size: 14px;
}

.tabla-reservas tbody tr:hover {
  background-color: #f9f9f9;
}

.ordenar-icon {
  font-size: 19px;
  margin-left: 4px;
  color: #999;
}

.eliminar {
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

.eliminar:hover {
  color: #e74c3c;
}
</style>
