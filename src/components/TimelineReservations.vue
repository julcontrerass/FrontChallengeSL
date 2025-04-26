<template>
  <div>
      <h3 class="timeline-title">Agenda de Reservas</h3>
      
      <div class="legend">
          <div class="legend-item">
              <span class="legend-box available"></span>
              <span>Disponible</span>
          </div>
          <div class="legend-item">
              <span class="legend-box reserved"></span>
              <span>Reservado</span>
          </div>
          <div class="legend-item">
              <span class="legend-box my-reservation"></span>
              <span>Mi Reserva Actual</span>
          </div>
      </div>
      
      <div class="timeline-container">
          <!-- Timeline -->
          <div id="visualization">
            <!-- Botón izquierdo -->
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
export default {
  name: 'TimelineReservations',
  props: {
      reservations: {
          type: Array,
          default: () => []
      }
  },
  data() {
    return {
      timeline: null,
      items: null
    }
  },
  mounted() {
    // Cargar dependencias e inicializar el timeline
    this.loadDependencies().then(() => {
      this.initializeTimeline();
    }).catch(error => {
      console.error('Error al cargar las dependencias:', error);
    });
  },
  watch: {
      reservations: {
          handler() {
              this.updateItems();
          },
          deep: true
      }
  },
  methods: {
      loadDependencies() {
        return new Promise((resolve, reject) => {
          // Verificar si ya están cargadas las dependencias
          if (window.moment && window.vis) {
            resolve();
            return;
          }

          // Cargar desde la carpeta public/vis
          const momentScript = document.createElement('script');
          momentScript.src = '/vis/moment.min.js';
          momentScript.onload = () => {
            const visScript = document.createElement('script');
            visScript.src = '/vis/vis-timeline-graph2d.min.js';
            visScript.onload = resolve;
            visScript.onerror = reject;
            document.head.appendChild(visScript);
          };
          momentScript.onerror = reject;
          document.head.appendChild(momentScript);
        });
      },
      initializeTimeline() {
          // Verificar disponibilidad de dependencias
          if (!window.moment || !window.vis || !window.vis.DataSet || !window.vis.Timeline) {
            console.error('Las dependencias necesarias no están disponibles');
            return;
          }
          
          // Crear DataSet
          const items = new window.vis.DataSet();
          
          // Configurar opciones del timeline
          const options = {
              stack: false,
              start: window.moment().startOf('day').add(8, 'hours').toDate(),
              end: window.moment().startOf('day').add(18, 'hours').toDate(),
              editable: false,
              zoomable: false,
              moveable: true
          };
          
          // Crear timeline
          const container = document.getElementById('visualization');
          if (!container) {
            console.error('No se encontró el contenedor del timeline');
            return;
          }
          
          const timeline = new window.vis.Timeline(container, items, options);
          
          // Configurar botones de navegación
          const fitButton = document.getElementById('fit');
          if (fitButton) {
            fitButton.addEventListener('click', () => {
              timeline.fit();
            });
          }
          
          const moveLeftButton = document.getElementById('moveLeft');
          if (moveLeftButton) {
            moveLeftButton.addEventListener('click', () => {
              const range = timeline.getWindow();
              const interval = range.end - range.start;
              timeline.setWindow({
                  start: new Date(range.start.valueOf() - interval * 0.2),
                  end: new Date(range.end.valueOf() - interval * 0.2)
              });
            });
          }
          
          const moveRightButton = document.getElementById('moveRight');
          if (moveRightButton) {
            moveRightButton.addEventListener('click', () => {
              const range = timeline.getWindow();
              const interval = range.end - range.start;
              timeline.setWindow({
                  start: new Date(range.start.valueOf() + interval * 0.2),
                  end: new Date(range.end.valueOf() + interval * 0.2)
              });
            });
          }
          
          // Guardar referencias
          this.timeline = timeline;
          this.items = items;
          
          // Cargar items iniciales
          this.updateItems();
          
          console.log('Timeline inicializado correctamente');
      },
      updateItems() {
          if (!this.items || !this.timeline) {
            console.warn('Timeline o items no inicializados');
            return;
          }
          
          // Limpiar items existentes
          this.items.clear();
          
          // Verificar si hay reservas para mostrar
          if (!this.reservations || this.reservations.length === 0) {
            console.log('No hay reservas para mostrar');
            return;
          }
          
          // Agregar las reservas al timeline
          const reservationItems = this.reservations.map(reservation => {
              let startTime, endTime;
              
              if (reservation.start) {
                  startTime = window.moment(reservation.start);
              } else {
                  startTime = window.moment(`${reservation.date} ${reservation.startTime}`);
              }
              
              if (reservation.end) {
                  endTime = window.moment(reservation.end);
              } else {
                  endTime = window.moment(`${reservation.date} ${reservation.endTime}`);
              }
              
              return {
                  id: reservation.id,
                  content: reservation.content || reservation.name,
                  start: startTime.toDate(),
                  end: endTime.toDate(),
                  // Aplicar clase según tipo de reserva
                  className: reservation.id === 1 ? 'my-reservation-item' : 'reserved-item'
              };
          });
          
          this.items.add(reservationItems);
          
          // Ajustar vista si hay items
          if (reservationItems.length > 0) {
              this.timeline.fit();
          }
      }
  }
}
</script>

<style scoped>
/* Contenedor del timeline */
.timeline-container {
  position: relative;
  width: 100%;
  height: 150px;
}

/* Estilo para botones laterales */
.side-button {
  position: absolute;
  top: 0;
  height: 100%;
  width: 40px;
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
}

.side-button.left {
  left: 0;
  background-color: rgba(245, 245, 245, 0.7);
  border-right: 1px solid #e0e0e0;
}

.side-button.right {
  right: 0;
  background-color: rgba(245, 245, 245, 0.7);
  border-left: 1px solid #e0e0e0;
}

/* Estilo para botón inferior */
.bottom-button {
  position: absolute;
  bottom: -40px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
}

/* Estilo para iconos */
.buttons-menu {
  cursor: pointer;
  color: rgba(55, 54, 54, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.2s;
}

/* Estilo para botón home */
.bottom-button .buttons-menu {
  background-color: #f5f5f5;
  padding: 8px;
  border-radius: 50%;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.buttons-menu:hover {
  color: rgba(0, 0, 0, 0.8);
}

.bottom-button .buttons-menu:hover {
  background-color: #e0e0e0;
}

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

.legend-box.available {
  background-color: #f8f8f8;
  border: 1px solid #e0e0e0;
}

.legend-box.reserved {
  background-color: #fde9a8;
  border: 1px solid #f8d678;
}

.legend-box.my-reservation {
  background-color: #b5f0c4;
  border: 1px solid #8de89f;
}

/* Estilos para items del timeline */
:deep(.my-reservation-item) {
  background-color: #b5f0c4 !important;
  border: 1px solid #8de89f !important;
}

:deep(.reserved-item) {
  background-color: #fde9a8 !important;
  border: 1px solid #f8d678 !important;
}
</style>