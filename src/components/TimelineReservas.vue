<template>
  <div>
      <h3 class="timeline-title">Agenda de Reservas</h3>
      
      <div class="legend">
          <div class="legend-item">
              <span class="legend-box disponible"></span>
              <span>Disponible</span>
          </div>
          <div class="legend-item">
              <span class="legend-box reservado"></span>
              <span>Reservado</span>
          </div>
          <div class="legend-item">
              <span class="legend-box mi-reserva"></span>
              <span>Mi Reserva Actual</span>
          </div>
      </div>
      
      <div class="timeline-container">
          <!-- Botón izquierdo
          <div class="side-button left">
              <i class="material-icons dp48 buttons-menu" id="moveLeft">arrow_back</i>
          </div> -->
          
          <!-- Timeline -->
          <div id="visualization">

            <div class="side-button left">
              <i class="material-icons dp48 buttons-menu" id="moveLeft">arrow_back</i>
          </div>
          <!-- Botón derecho -->
          <div class="side-button right">
              <i class="material-icons dp48 buttons-menu" id="moveRight">arrow_forward</i>
          </div>
          
          <!-- Botón home centrado abajo -->
          <div class="bottom-button">
              <i class="material-icons dp48 buttons-menu" id="fit">home</i>
          </div>
          </div>
          
          
      </div>
  </div>
</template>

<script>
import moment from "../assets/vis/moment.js";
import { Timeline, DataSet } from "../assets/vis/vis.js";

export default {
  name: 'TimelineReservas',
  props: {
      reservas: {
          type: Array,
          default: () => []
      }
  },
  mounted() {
      // Cargar los íconos de Material Design
      if (!document.getElementById('material-icons-css')) {
          const link = document.createElement('link');
          link.id = 'material-icons-css';
          link.rel = 'stylesheet';
          link.href = 'https://fonts.googleapis.com/icon?family=Material+Icons';
          document.head.appendChild(link);
      }

      // Cargar los estilos CSS necesarios
      if (!document.getElementById('style-css')) {
          const link = document.createElement('link');
          link.id = 'style-css';
          link.rel = 'stylesheet';
          link.type = 'text/css';
          link.href = '../assets/vis/style.css';
          document.head.appendChild(link);
      }

      if (!document.getElementById('full-style-css')) {
          const link = document.createElement('link');
          link.id = 'full-style-css';
          link.rel = 'stylesheet';
          link.type = 'text/css';
          link.href = '../assets/vis/full-style.css';
          document.head.appendChild(link);
      }

      this.initializeTimeline();
  },
  watch: {
      reservas: {
          handler() {
              this.updateItems();
          },
          deep: true
      }
  },
  methods: {
      initializeTimeline() {
          // Create a DataSet
          const items = new DataSet();
          
          // Configurar opciones del timeline con tamaño fijo sin zoom
          const options = {
              stack: false,
              start: moment().startOf('day').add(8, 'hours').toDate(),
              end: moment().startOf('day').add(18, 'hours').toDate(),
              editable: false,
              zoomable: false, // Deshabilitar zoom
              moveable: true   // Mantener la capacidad de moverse
          };
          
          // Crear timeline
          const container = document.getElementById('visualization');
          const timeline = new Timeline(container, items, options);
          
          // Agregar los eventos de los botones
          document.getElementById('fit').addEventListener('click', () => {
              timeline.fit();
          });
          
          document.getElementById('moveLeft').addEventListener('click', () => {
              const range = timeline.getWindow();
              const interval = range.end - range.start;
              timeline.setWindow({
                  start: new Date(range.start.valueOf() - interval * 0.2),
                  end: new Date(range.end.valueOf() - interval * 0.2)
              });
          });
          
          document.getElementById('moveRight').addEventListener('click', () => {
              const range = timeline.getWindow();
              const interval = range.end - range.start;
              timeline.setWindow({
                  start: new Date(range.start.valueOf() + interval * 0.2),
                  end: new Date(range.end.valueOf() + interval * 0.2)
              });
          });
          
          // Guardar referencias para usar más tarde
          this.timeline = timeline;
          this.items = items;
          
          // Cargar los items iniciales
          this.updateItems();
      },
      updateItems() {
          if (!this.items || !this.timeline) return;
          
          // Limpiar items existentes
          this.items.clear();
          
          // Agregar las reservas
          const reservaItems = this.reservas.map(reserva => {
              let startTime, endTime;
              
              if (reserva.start) {
                  startTime = moment(reserva.start);
              } else {
                  startTime = moment(`${reserva.fecha} ${reserva.horaInicio}`);
              }
              
              if (reserva.end) {
                  endTime = moment(reserva.end);
              } else {
                  endTime = moment(`${reserva.fecha} ${reserva.horaFin}`);
              }
              
              return {
                  id: reserva.id,
                  content: reserva.content || reserva.nombre,
                  start: startTime.toDate(),
                  end: endTime.toDate(),
                  // Agregar clase según el tipo de reserva (para la leyenda)
                  className: reserva.id === 1 ? 'mi-reserva-item' : 'reservado-item'
              };
          });
          
          this.items.add(reservaItems);
          
          // Ajustar la vista si hay items
          if (reservaItems.length > 0) {
              this.timeline.fit();
          }
      }
  }
}
</script>

<style scoped>
/* Los estilos se cargan desde los archivos CSS externos */
/* Añadir estilos adicionales específicos del componente */
.timeline-container {
  position: relative;
  width: 100%;
  height: 150px;
}


/* Eliminar el menú original con slider */
.menu {
  display: none;
}

/* Estilo para los botones laterales */
.side-button {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  z-index: 100;
}

.side-button.left {
  left: 5px;
}

.side-button.right {
  right: 5px;
}

/* Estilo para el botón inferior */
.bottom-button {
  position: absolute;
  bottom: -40px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
}

.buttons-menu {
  cursor: pointer;
  background-color: #f5f5f5;
  padding: 8px;
  border-radius: 50%;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.2s;
}

.buttons-menu:hover {
  background-color: #e0e0e0;
}

/* Estilos para la leyenda */
.timeline-title {
  margin-bottom: 10px;
  color: #555;
  font-size: 18px;
  font-weight: 500;
}

.legend {
  display: flex;
  justify-content: flex-end;
  gap: 20px;
  margin-bottom: 15px;
}

.legend-item {
  display: flex;
  align-items: center;
  font-size: 14px;
  color: #666;
}

.legend-box {
  display: inline-block;
  width: 14px;
  height: 14px;
  margin-right: 8px;
  border-radius: 3px;
}

.legend-box.disponible {
  background-color: #f8f8f8;
  border: 1px solid #e0e0e0;
}

.legend-box.reservado {
  background-color: #fde9a8;
  border: 1px solid #f8d678;
}

.legend-box.mi-reserva {
  background-color: #b5f0c4;
  border: 1px solid #8de89f;
}

/* Estilos para items en el timeline */
:deep(.mi-reserva-item) {
  background-color: #b5f0c4 !important;
  border: 1px solid #8de89f !important;
}

:deep(.reservado-item) {
  background-color: #fde9a8 !important;
  border: 1px solid #f8d678 !important;
}
</style>