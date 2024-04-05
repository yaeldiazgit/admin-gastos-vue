<script setup>
  import { ref, reactive, watch, computed, onMounted } from "vue";
  import Presupuesto from './components/Presupuesto.vue'
  import ControlPresupuesto from "./components/ControlPresupuesto.vue";
  import Modal from "./components/Modal.vue"
  import Gasto from "./components/Gasto.vue"
  import Filtros from "./components/Filtros.vue";
  import iconoNuevoGasto from "../src/assets/img/nuevo-gasto.svg";
  import { generarID } from "./helpers";

  const presupuesto = ref(0);
  const gasto = reactive({
    nombre: '',
    cantidad: '',
    categoria: '',
    id: null,
    fecha: Date.now()
  });
  const gastos = ref([]);
  const gastado = ref(0);
  const disponible = ref(0);
  const filtro = ref('');
  const modal = reactive({
    mostrar: false,
    animar: false
  });

  onMounted(() => {
    const presupuestoStorage = localStorage.getItem('presupuesto');
    const gastosStorage = localStorage.getItem('gastos');
    if(presupuestoStorage){
      presupuesto.value = Number(presupuestoStorage);
      disponible.value = Number(presupuestoStorage);
    }
    if(gastosStorage) {
      gastos.value = JSON.parse(gastosStorage);
    }
  })
  watch(gastos, () => {
    gastado.value = gastos.value.reduce((total, gastoInfo) => total + gastoInfo.cantidad,0);
    disponible.value = presupuesto.value - gastado.value;
    localStorage.setItem('gastos', JSON.stringify(gastos.value));
  }, {
    deep: true
  });

  watch(presupuesto, () => {
    localStorage.setItem('presupuesto', presupuesto.value);
  });


  const definirPresupuesto = (cantidad) => {
    presupuesto.value = cantidad;
    disponible.value = cantidad;
  }
  const mostrarModal = () => {
    modal.mostrar = true;
    setTimeout(() => {
      modal.animar = true;
    }, 300)
  }
  const ocultarModal = () => {
    modal.animar = false;
    setTimeout(() => {
      modal.mostrar = false;
      Object.assign(gasto, {
      nombre: '',
      cantidad: '',
      categoria: '',
      id: null,
      fecha: Date.now()
    });
    }, 300);
  }

  const añadirGasto = () => {
    const index = gastos.value.findIndex((gastoIndex) => gastoIndex.id === gasto.id);
    if(index < 0) {
      gastos.value.push({
      ...gasto,
      id: generarID()
    });
    } else {
      gastos.value[index] = {...gasto};
    }
    ocultarModal();
  }

  const editarGasto = (gastoEditar) =>  {
    Object.assign(gasto, gastoEditar);
    mostrarModal();
  }

  const eliminarGasto = () => {
    gastos.value = gastos.value.filter((gastoEliminar) => gastoEliminar.id !== gasto.id);
    ocultarModal();
  }
  
  const gastosFiltrados = computed(() => {
    if(filtro.value) {
      return gastos.value.filter((gastosFiltro) => gastosFiltro.categoria === filtro.value);
    } else {
      return gastos.value;
    }
  });

  const resetApp = () => {
    presupuesto.value = 0;
    gastos.value = [];
  }
</script>

<template>
  <div 
    :class="{fijar: modal.mostrar}"
  >
    <header>
      <h1>Planificador de Gastos</h1>
      <div class="contenedor-header contenedor sombra">
        <Presupuesto
          v-if="presupuesto === 0"
          @definir-presupuesto="definirPresupuesto"
        />
        <ControlPresupuesto
          v-else
          :presupuesto="presupuesto"
          :gastado="gastado"
          :disponible="disponible"
          @reset-app="resetApp"
        />
        
      </div>
    </header>

    <main  v-if="presupuesto > 0">
      <Filtros 
        v-model:filtro="filtro"
      />
      <div class="listado-gastos contenedor">
        <h2>
          {{ gastosFiltrados.length > 0 ? 'Gastos': 'No hay gastos' }}
        </h2>
        <Gasto
          v-for="gasto in gastosFiltrados" 
          :key="gasto.id"
          :gasto="gasto"
          @editar-gasto="editarGasto"
        />
      </div>
      <div class="crear-gasto">
        <img :src="iconoNuevoGasto" alt="nuevo gasto" @click="mostrarModal">
      </div>
      <Modal
        v-if="modal.mostrar"
        v-model:nombre="gasto.nombre"
        v-model:cantidad="gasto.cantidad"
        v-model:categoria="gasto.categoria"
        :modal="modal"
        :idGasto="gasto.id"
        :disponible="disponible"
        @ocultar-modal="ocultarModal"
        @añadir-gasto="añadirGasto"
        @eliminar-gasto="eliminarGasto"
      />
    </main>
  </div>
</template>

<style>
  :root {
    --azul: #3b82f6;
    --blanco: #FFF;
    --gris-claro: #F5F5F5;
    --gris: #94a3b8;
    --gris-oscuro: #64748b;
    --negro: #000;
  }
  html {
    font-size: 62.5%;
    box-sizing: border-box;
  }
  *,
  *::before,
  *::after {
    box-sizing: inherit;
  }
  body {
    font-size: 1.6rem;
    font-family: "Lato", sans-serif;
    background-color: var(--gris-claro);
  }
  h1{
    font-size: 4rem;
  }
  h2{
    font-size: 3rem;
  }
  .fijar {
    overflow: hidden;
    height: 100vh;
  }
  header{
    background-color: var(--azul);
  }
  header h1 {
    padding: 3rem 0;
    margin: 0;
    color: var(--blanco);
    text-align: center;
  }
  .contenedor {
    width: 90%;
    max-width: 80rem;
    margin: 0 auto;
  }
  .contenedor-header {
    margin-top: -5rem;
    transform: translateY(5rem);
    padding: 5rem;
  }
  .sombra {
    box-shadow: 0px 10px 15px -3px rgba(0,0,0,0.1);
    background-color: var(--blanco);
    border-radius: 1.2rem;
    padding: 5rem;
  }

  .crear-gasto {
    position: fixed;
    bottom: 5rem;
    right: 5rem;
  }
  .crear-gasto img {
    width: 5rem;
    cursor: pointer;
  }
  .listado-gastos {
    margin-top: 10rem;
  }
  .listado-gastos h2 {
    font-weight: 900;
    color: var(--gris-oscuro);
  }
</style>
