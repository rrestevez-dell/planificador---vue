<script setup>
  // Importando dependencias
  import { ref, reactive, watch, computed, onMounted } from 'vue';

  // Importando funciones creadas en JavaScript 
  import { seleccionarTexto, generarId } from "./helpers/index";

  //Importando las Imagenes, Gif, ...
  import iconoNuevoGasto from './assets/img/nuevo-gasto.svg'

  // Importando los componentes
  import Presupuesto from './components/Presupuesto.vue'
  import Filtros from './components/Filtros.vue'
  import ControlPresuPuesto from './components/ControlPreaupuesto.vue'
  import Modal from './components/Modal.vue'
  import Gasto from './components/Gasto.vue';
  
  //----Lógica-----

  const modal = reactive({
    mostrar:false,
    animar: false,
  })
  const presupuesto = ref(0)
  const disponible = ref(0)
  const gastado = ref(0)
  const filtro = ref('')

  const gasto = reactive({
    nombre: '',
    cantidad: '',
    categoria: '',
    id: null,
    fecha: Date.now()
  })

  const gastos = ref([])

  watch(gastos, () => {
    const totalGastado = gastos.value.reduce((total, gasto)  => gasto.cantidad + total, 0)
    gastado.value = totalGastado
    disponible.value = presupuesto.value - gastado.value

    localStorage.setItem('gastos', JSON.stringify(gastos.value))
  }, {
    deep: true
  })

  watch(modal, () => {
    if (!modal.mostrar) {
      //Reiniciar el objeto
      reiniciarStateGasto()
    }
  }, {
    deep: true
  })

  watch(presupuesto, () => {
    localStorage.setItem('presupuesto', presupuesto.value)
  })

  onMounted(() => {
    const presupuestoStorange = localStorage.getItem('presupuesto')
    if (presupuestoStorange) {
      presupuesto.value = Number(presupuestoStorange)
      disponible.value = Number(presupuestoStorange)
    }
    const gastosStorange = localStorage.getItem('gastos')
    if (gastosStorange) {
      gastos.value = JSON.parse(gastosStorange)
    }
  })

  const definirPresupuesto =  (cantidad) => {
    presupuesto.value = cantidad
    disponible.value = cantidad
  }

  const mostrarModal = () => {
    modal.mostrar = true
    setTimeout(() => {
      modal.animar = true
    }, 300)
  }

  const ocultarModal = () => {
    modal.animar = false
    setTimeout(() => {
      modal.mostrar = false
    }, 300)
  }

  const guardarGastos = () => {
    if (gasto.id) {
      //Estamos editando
      const { id } = gasto
      const i = gastos.value.findIndex(gasto  => gasto.id === id)
      gastos.value[i] = {...gasto}
    }else {
      //Es un registro nuevo 
      gastos.value.push({
      ...gasto,
      id: generarId(),
    })
    }
    ocultarModal()
    //Reiniciando el Objeto
    reiniciarStateGasto()
  }

  const reiniciarStateGasto = () => {
    //Reiniciando el Objeto
    Object.assign(gasto, {
      nombre: '',
      cantidad: '',
      categoria: '',
      id: null,
      fecha: Date.now()
    })
  }

  const seleccionarGasto = id => {
    const gastoEditar = gastos.value.filter(gasto  => gasto.id === id)[0]
    Object.assign(gasto, gastoEditar);
    mostrarModal()
  }

  const eliminarGasto = () => {
    if (confirm('Eliminar?')) {
      gastos.value = gastos.value.filter(gastoState  => gastoState.id !== gasto.id)
      ocultarModal()
    }
  }
  
  const gastosFiltrados = computed(() => {
    if(filtro.value) {
      return gastos.value.filter(gasto => gasto.categoria === filtro.value)
    }
    return gastos.value
  })

  const resetApp = () => {
    if (confirm('Eliminar?')) {
      gastos.value = []
      presupuesto.value = 0
    }
  }

</script>

<template>
  <div
    :class="{fijar: modal.mostrar}"
  >
    <header>
      <h1>Planificador de Gastos</h1>
      <div class="contenedor-heder contenedor sombra">
        <Presupuesto
          v-if="presupuesto === 0"
          @seleccionar-texto="seleccionarTexto"
          @definir-presupuesto="definirPresupuesto"
          
        />
 
        <ControlPresuPuesto
          v-else
          :presupuesto="presupuesto"
          :disponible="disponible"
          :gastado="gastado"
          @reset-app="resetApp"
        />
        
      </div>
    </header>

    <main v-if="presupuesto > 0">
      <Filtros
        v-model:filtro="filtro"
      />
      <div class="listado-gastos contenedor">
        <h2>{{ gastosFiltrados.length > 0 ? 'Gastos' :'No hay Gastos' }}</h2>
        <Gasto
          v-for="gasto in gastosFiltrados"
          :key="gasto.id"
          :gasto="gasto"
          @seleccionar-gasto="seleccionarGasto"
        />
      </div>

      <div class="crear-gasto">
        <img 
          :src="iconoNuevoGasto" 
          alt="Icono Nuevo Gasto"
          @click="mostrarModal"
          >
      </div>

       <Modal
        v-if="modal.mostrar"
        @ocultar-modal="ocultarModal"
        @seleccionar-texto="seleccionarTexto"
        @guardar-gastos="guardarGastos"
        @eliminar-gasto="eliminarGasto"
        :modal="modal"
        :disponible="disponible"
        :id="gasto.id"
        v-model:nombre="gasto.nombre"
        v-model:cantidad="gasto.cantidad"
        v-model:categoria="gasto.categoria"
       />

    </main>
  </div>
</template>

<style>
  :root {
    --azul: #3b82f6;
    --blanco: #fff;
    --gris-claro: #f5f5f5;
    --gris: #94a3b8;
    --gris-oscuro: #64748b;
    --negro: #000;
  }
  html {
    font-size: 62.5%;
    box-sizing: border-box;
  }
  *,
  *:before,
  *:after {
    box-sizing: inherit;
  }
  body {
    font-size: 1.6rem;
    font-family: "Lato";
    background-color: var(--gris-claro);
  }
  h1 {
    font-size: 4rem;
  }
  h2 {
    font-size: 3rem;
  }

  .fijar {
    overflow: hidden;
    height: 100vh;
  }

  header {
    background-color: var(--azul);
  }
  header h1 {
    padding: 3rem 0;
    margin:  0;
    color: var(--blanco);
    text-align: center;
  }
  .contenedor {
    width: 98%;
    max-width: 80rem;
    margin: 0 auto;
  }
  .contenedor-heder {
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
    text-align: center;
  }

  .listado-gastos {
    font-weight: 900;
    color: var(--gris-oscuro);
  }
 
</style>./components/Resultado.vue