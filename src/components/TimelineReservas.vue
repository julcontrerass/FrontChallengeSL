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
    
    <div id="timeline-container">
      <button class="nav-button prev" @click="moveTimeline(0.2)">
        <i class="fas fa-chevron-left"></i>
      </button>
      
      <div id="visualization" ref="timeline"></div>
      
      <button class="nav-button next" @click="moveTimeline(-0.2)">
        <i class="fas fa-chevron-right"></i>
      </button>
    </div>
  </div>
</template>

<script>
import moment from "../assets/vis/moment.js";
import { Timeline, DataSet } from "../assets/vis/vis.js";

export default {
  name: "TimelineReservas",
  props: {
    reservas: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      timeline: null,
      items: new DataSet([]),
      options: {
        orientation: {
          axis: "top",
          item: "top"
        },
        zoomable: false,
        moveable: true,
        margin: {
          item: 10
        },
        stack: false,
        start: moment().startOf('day').add(8, 'hours'),
        end: moment().startOf('day').add(18, 'hours'),
        showMajorLabels: true,
        format: {
          minorLabels: function(date) {
            const momentDate = moment.isMoment(date) ? date : moment(date);
            const minutes = momentDate.minutes();
            return minutes !== 0 ? momentDate.format("mm") : '';
          },
          majorLabels: function(date) {
            const momentDate = moment.isMoment(date) ? date : moment(date);
            if (momentDate.minutes() === 0) {
              return momentDate.format("HH"); // Formato 24h sin minutos
            }
            return '';
          }
        },
        timeAxis: { scale: 'minute', step: 15 },
        template: function(item) {
          const isCurrentUser = item.currentUser;
          const colorClass = isCurrentUser ? 'mi-reserva-item' : 'reservado-item';
          return `<div class="timeline-item ${colorClass}">${item.content}</div>`;
        }
      }
    };
  },
  mounted() {
    moment.locale('es'); // Añadir locale español
    this.$nextTick(() => {
      this.initializeTimeline();
      this.updateTimelineItems();
    });
    
    if (!document.getElementById('fontawesome-css')) {
      const link = document.createElement('link');
      link.id = 'fontawesome-css';
      link.rel = 'stylesheet';
      link.href = 'https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css';
      document.head.appendChild(link);
    }
    
    // Cargar los estilos de full-style.css para vis.js si es necesario
    if (!document.getElementById('vis-timeline-css')) {
      const link = document.createElement('link');
      link.id = 'vis-timeline-css';
      link.rel = 'stylesheet';
      link.href = '../assets/vis/full-style.css'; // Ajusta la ruta según tu estructura
      document.head.appendChild(link);
    }
  },
  watch: {
    reservas: {
      handler() {
        this.$nextTick(() => {
          this.updateTimelineItems();
        });
      },
      deep: true
    }
  },
  methods: {
    initializeTimeline() {
      try {
        if (!this.$refs.timeline) {
          console.error('No se encontró el elemento DOM para el timeline');
          return;
        }
        
        // Comprobar que las dependencias estén cargadas
        if (!Timeline || !DataSet || !moment) {
          console.error('Las dependencias (Timeline, DataSet o moment) no están cargadas correctamente');
          return;
        }
        
        this.timeline = new Timeline(this.$refs.timeline, this.items, this.options);
        
        // Establecer vista inicial centrada en el horario laboral
        const today = moment().startOf('day');
        this.timeline.setWindow(
          today.clone().add(9, 'hours').toDate(),
          today.clone().add(15, 'hours').toDate()
        );
        
        console.log('Timeline inicializado correctamente');
      } catch (error) {
        console.error('Error al inicializar el timeline:', error);
      }
    },
    updateTimelineItems() {
      if (!this.timeline) {
        console.error('El timeline no está inicializado');
        this.initializeTimeline(); // Intentar inicializar de nuevo
        if (!this.timeline) return; // Si sigue sin inicializarse, salir
      }
      
      try {
        // Limpiar items actuales
        this.items.clear();
        
        // Añadir items basados en las reservas props
        const mappedItems = this.reservas.map(reserva => {
          // Asegurarse de que las fechas sean objetos moment válidos
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
            // Supongamos que la primera reserva es la del usuario actual (para demo)
            currentUser: reserva.id === 1
          };
        });
        
        this.items.add(mappedItems);
        
        // Ajustar la vista para mostrar todas las reservas
        if (mappedItems.length > 0) {
          this.resetZoom();
        }
      } catch (error) {
        console.error('Error al actualizar los items del timeline:', error);
      }
    },
    moveTimeline(percentage) {
      if (!this.timeline) {
        console.error('El timeline no está inicializado o es null');
        return;
      }
      
      try {
        const range = this.timeline.getWindow();
        const interval = range.end - range.start;
        this.timeline.setWindow({
          start: new Date(range.start.valueOf() - interval * percentage),
          end: new Date(range.end.valueOf() - interval * percentage)
        });
      } catch (error) {
        console.error('Error al mover el timeline:', error);
      }
    },
    resetZoom() {
      if (!this.timeline) {
        console.error('El timeline no está inicializado');
        return;
      }
      
      try {
        if (this.reservas.length > 0) {
          // Encontrar la primera y última reserva para ajustar la vista
          const times = this.reservas.flatMap(r => {
            const startTime = r.start ? moment(r.start) : moment(`${r.fecha} ${r.horaInicio}`);
            const endTime = r.end ? moment(r.end) : moment(`${r.fecha} ${r.horaFin}`);
            return [startTime, endTime];
          });
          
          // Agregar margen antes y después
          const minTime = moment.min(times).subtract(1, 'hour');
          const maxTime = moment.max(times).add(1, 'hour');
          
          this.timeline.setWindow(
            minTime.toDate(),
            maxTime.toDate()
          );
        } else {
          // Vista predeterminada si no hay reservas
          const today = moment().startOf('day');
          this.timeline.setWindow(
            today.clone().add(9, 'hours').toDate(),
            today.clone().add(17, 'hours').toDate()
          );
        }
      } catch (error) {
        console.error('Error al resetear el zoom:', error);
      }
    }
  }
};
</script>

<style scoped>
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

#timeline-container {
  position: relative;
  display: flex;
  align-items: center;
  margin-bottom: 40px;
}

#visualization {
  flex: 1;
  height: 180px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: white;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

.nav-button {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 40px;
  width: 40px;
  border-radius: 50%;
  border: 1px solid #ddd;
  background: white;
  color: #666;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.2s;
}

.nav-button:hover {
  background: #f5f5f5;
  border-color: #ccc;
}

.nav-button.prev {
  margin-right: 10px;
}

.nav-button.next {
  margin-left: 10px;
}

/* Estilos para items en el timeline */
:deep(.timeline-item) {
  padding: 5px 10px;
  border-radius: 4px;
  font-size: 13px;
  height: 100%;
}

:deep(.reservado-item) {
  background-color: #fde9a8;
  border: 1px solid #f8d678;
}

:deep(.mi-reserva-item) {
  background-color: #b5f0c4;
  border: 1px solid #8de89f;
}

:deep(.vis-timeline) {
  border: none;
  font-family: "Montserrat", sans-serif;
}

:deep(.vis-time-axis .vis-text) {
  color: #666;
  font-weight: 500;
}

/* Estilos para horas completas (etiquetas principales) */
:deep(.vis-time-axis .vis-text.vis-major) {
  font-size: 16px;
  font-weight: bold;
  color: #000000;
  margin-top: 5px;
}

/* Estilos para minutos (etiquetas secundarias) */
:deep(.vis-time-axis .vis-text.vis-minor) {
  font-size: 12px;
  color: #888;
}

/* Ajustar espacio vertical para etiquetas */
:deep(.vis-time-axis .vis-text) {
  padding-top: 5px;
  padding-bottom: 5px;
}

:deep(.vis-grid.vis-vertical) {
  border-left: 1px solid #f0f0f0;
}

:deep(.vis-panel.vis-center),
:deep(.vis-panel.vis-left),
:deep(.vis-panel.vis-right),
:deep(.vis-panel.vis-top),
:deep(.vis-panel.vis-bottom) {
  border: none;
}

:deep(.vis-time-axis .vis-text.vis-major) {
  font-size: 14px !important;
  color: #333 !important;
  font-weight: bold;
}

:deep(.vis-time-axis .vis-text.vis-minor) {
  font-size: 12px !important;
  color: #666 !important;
}
</style>