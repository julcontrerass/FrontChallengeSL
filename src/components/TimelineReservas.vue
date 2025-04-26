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
  name: 'TimelineReservas',
  props: {
      reservas: {
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
    // Importar las dependencias y luego inicializar el timeline
    this.cargarDependencias().then(() => {
      this.initializeTimeline();
    }).catch(error => {
      console.error('Error al cargar las dependencias:', error);
    });
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
      cargarDependencias() {
        return new Promise((resolve, reject) => {
          // Verificar si ya están cargadas
          if (window.moment && window.vis) {
            resolve();
            return;
          }

          // Si no están cargadas, intentar cargar desde la carpeta public/vis
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
          // Comprobar si las dependencias están disponibles
          if (!window.moment || !window.vis || !window.vis.DataSet || !window.vis.Timeline) {
            console.error('Las dependencias necesarias no están disponibles');
            return;
          }
          
          // Crear un DataSet
          const items = new window.vis.DataSet();
          
          // Configurar opciones del timeline con tamaño fijo sin zoom
          const options = {
              stack: false,
              start: window.moment().startOf('day').add(8, 'hours').toDate(),
              end: window.moment().startOf('day').add(18, 'hours').toDate(),
              editable: false,
              zoomable: false, // Deshabilitar zoom
              moveable: true   // Mantener la capacidad de moverse
          };
          
          // Crear timeline
          const container = document.getElementById('visualization');
          if (!container) {
            console.error('No se encontró el contenedor del timeline');
            return;
          }
          
          const timeline = new window.vis.Timeline(container, items, options);
          
          // Agregar los eventos de los botones
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
          
          // Guardar referencias para usar más tarde
          this.timeline = timeline;
          this.items = items;
          
          // Cargar los items iniciales
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
          
          // Verificar si tenemos reservas para mostrar
          if (!this.reservas || this.reservas.length === 0) {
            console.log('No hay reservas para mostrar');
            return;
          }
          
          // Agregar las reservas
          const reservaItems = this.reservas.map(reserva => {
              let startTime, endTime;
              
              if (reserva.start) {
                  startTime = window.moment(reserva.start);
              } else {
                  startTime = window.moment(`${reserva.fecha} ${reserva.horaInicio}`);
              }
              
              if (reserva.end) {
                  endTime = window.moment(reserva.end);
              } else {
                  endTime = window.moment(`${reserva.fecha} ${reserva.horaFin}`);
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
/* Estilos adicionales específicos del componente */
.timeline-container {
  position: relative;
  width: 100%;
  height: 150px;
}

/* Eliminar el menú original con slider */
.menu {
  display: none;
}

/* Estilo para los botones laterales - MODIFICADO para ser rectangulares */
.side-button {
  position: absolute;
  top: 0;
  height: 100%; /* Ocupa toda la altura del timeline */
  width: 40px; /* Ancho del botón */
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

/* Estilo para el botón inferior */
.bottom-button {
  position: absolute;
  bottom: -40px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
}

/* Estilo para los íconos dentro de los botones */
.buttons-menu {
  cursor: pointer;
  color: rgba(55, 54, 54, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.2s;
}

/* Estilo específico para el botón home que sigue siendo redondo */
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