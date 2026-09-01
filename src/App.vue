<template>
  <div class="app">
    <!-- ENCABEZADO -->
    <q-header class="cab">
      <q-toolbar>
        <q-toolbar-title class="flex items-center gap-2">
          <q-icon name="build_circle" size="32px" color="primary" />
          <span class="text-weight-bold text-gradient">Servicio Técnico</span>
        </q-toolbar-title>

        <q-btn
          unelevated
          label="Nuevo Servicio"
          icon="add"
          color="primary"
          class="btn-primary-glow"
          @click="nuevo"
        />
      </q-toolbar>
    </q-header>

    <!-- CONTENIDO PRINCIPAL -->
    <main class="cont">
      <!-- 1. HEADER HERO / PORTADA -->
      <div class="header-section header-banner q-mb-xl">
        <q-img
          src="https://images.unsplash.com/photo-1581092160607-ee22621dd758?auto=format&fit=crop&w=1200&q=80"
          height="220px"
          class="rounded-borders banner-img"
        >
          <div class="absolute-full flex flex-center text-center header-overlay">
            <div class="hero-title-container">
              <h1 class="hero-title text-gradient-hero">
                Servicios del Taller Técnico
              </h1>
              <div class="hero-divider"></div>
              <div class="sub-pill">
                <q-icon name="precision_manufacturing" size="18px" color="primary" class="q-mr-xs" />
                <span class="sub-text">Control inteligente de reparaciones y garantías</span>
              </div>
            </div>
          </div>
        </q-img>
      </div>

      <!-- 2. MÉTRICAS (UBICACIÓN SUPERIOR EN 3 COLUMNAS REPARTIDAS) -->
      <div class="row q-col-gutter-lg q-mb-xl">
        <div class="col-12 col-sm-4">
          <div class="metric-card metric-card-img">
            <q-img
              src="https://images.unsplash.com/photo-1518770660439-4636190af475?auto=format&fit=crop&w=300&q=80"
              class="metric-bg-img"
            />
            <div class="metric-content">
              <div class="metric-icon bg-blue-soft">
                <q-icon name="developer_board" color="primary" size="24px" />
              </div>
              <div class="metric-data">
                <span class="metric-title">Total Registros</span>
                <strong class="metric-value">{{ lista.length }}</strong>
              </div>
            </div>
          </div>
        </div>

        <div class="col-12 col-sm-4">
          <div class="metric-card">
            <div class="metric-icon bg-orange-soft">
              <q-icon name="handyman" color="warning" size="24px" />
            </div>
            <div class="metric-data">
              <span class="metric-title">En Reparación</span>
              <strong class="metric-value">{{ contarReparaciones() }}</strong>
            </div>
          </div>
        </div>

        <div class="col-12 col-sm-4">
          <div class="metric-card">
            <div class="metric-icon bg-red-soft">
              <q-icon name="account_balance_wallet" color="negative" size="24px" />
            </div>
            <div class="metric-data">
              <span class="metric-title">Pagos Pendientes</span>
              <strong class="metric-value">{{ contarPendientes() }}</strong>
            </div>
          </div>
        </div>
      </div>

      <!-- 3. SECCIÓN DE TARJETAS CON DISPOSICIÓN DINÁMICA CENTRADA -->
      <div class="row q-col-gutter-lg justify-center">
        <!-- REGISTROS EXISTENTES -->
        <div
          v-for="servicio in lista"
          :key="servicio.id"
          class="col-12 col-md-6 col-lg-4"
        >
          <q-card
            class="tar overflow-hidden full-height"
            :class="{
              'border-pago-pendiente': servicio.pago === 'Pendiente',
              'border-pago-abono': servicio.pago === 'Abono'
            }"
          >
            <!-- CABECERA CON IMAGEN -->
            <q-img
              :src="servicio.imagen || obtenerImagenPorDefecto(servicio.reparacion)"
              height="140px"
            >
              <div class="absolute-bottom bg-transparent text-white flex items-center justify-between q-px-sm q-py-xs">
                <q-badge :color="colorEquipo(servicio.estado)" class="badge-custom">
                  <q-icon :name="icono(servicio.estado)" size="12px" class="q-mr-xs" />
                  {{ servicio.estado }}
                </q-badge>

                <q-badge :color="colorPago(servicio.pago)" class="badge-custom">
                  <q-icon name="attach_money" size="12px" class="q-mr-xs" />
                  {{ servicio.pago }}
                </q-badge>
              </div>
            </q-img>

            <q-card-section class="q-pb-none">
              <!-- CLIENTE & EQUIPO -->
              <div class="cli q-mt-xs">
                <q-avatar size="48px" class="avatar-glow">
                  <q-icon name="person" size="28px" color="white" />
                </q-avatar>
                <div>
                  <div class="nom">{{ servicio.cliente }}</div>
                  <div class="eq">
                    <q-icon name="smartphone" size="16px" class="q-mr-xs" />
                    {{ servicio.marca }} {{ servicio.modelo }}
                  </div>
                </div>
              </div>

              <q-separator class="q-my-md divider-dark" />

              <!-- DETALLES -->
              <div class="info-grid">
                <p>
                  <q-icon name="build" />
                  <span><b>Reparación:</b> {{ servicio.reparacion }}</span>
                </p>
                <p>
                  <q-icon name="badge" />
                  <span><b>Técnico:</b> {{ servicio.tecnico }}</span>
                </p>
                <p>
                  <q-icon name="event" />
                  <span><b>Fecha:</b> {{ servicio.fecha }} - {{ servicio.hora }}</span>
                </p>
                <p>
                  <q-icon name="payments" />
                  <span><b>Precio:</b> ${{ precio(servicio.precio) }} ({{ servicio.metodo }})</span>
                </p>
              </div>

              <!-- CALIFICACIÓN -->
              <div 
                v-if="servicio.estado === 'Recibido' || servicio.estado === 'Entregado'" 
                class="cal q-mt-md"
              >
                <div class="text-caption text-weight-bold text-grey-4 q-mb-xs">
                  Calificación del servicio:
                </div>
                <div class="flex items-center gap-2">
                  <q-rating
                    v-model="servicio.calificacion"
                    size="24px"
                    color="warning"
                    icon="star_border"
                    icon-selected="star"
                  />
                </div>
              </div>

              <!-- OBSERVACIONES -->
              <div v-if="servicio.obs" class="obs">
                <b>Observaciones:</b>
                <p>{{ servicio.obs }}</p>
              </div>
            </q-card-section>

            <!-- BOTONES DE ACCIÓN -->
            <q-card-actions align="right" class="q-pt-md">
              <q-btn
                flat
                dense
                color="primary"
                icon="edit"
                label="EDITAR"
                @click="editarServicio(servicio)"
              />
              <q-btn
                flat
                dense
                color="negative"
                icon="delete"
                label="ELIMINAR"
                @click="eliminar(servicio.id)"
              />
            </q-card-actions>
          </q-card>
        </div>

        <!-- TARJETA PRINCIPAL PARA REGISTRAR (ADAPTABLE SEGÚN SI HAY O NO REGISTROS) -->
        <div 
          class="col-12"
          :class="lista.length === 0 ? 'col-md-8 col-lg-6' : 'col-md-6 col-lg-4'"
        >
          <div 
            class="card-nuevo-registro" 
            :class="{ 'empty-state-card': lista.length === 0 }"
            @click="nuevo"
          >
            <div class="nuevo-content">
              <div class="btn-add-circle">
                <q-icon name="add" size="36px" color="white" />
              </div>
              <h3 class="text-weight-bold q-mt-md q-mb-xs text-white">Registrar Nuevo Equipo</h3>
              <p class="text-grey-4">Haz clic aquí para agregar una nueva orden de servicio técnico al taller.</p>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- FORMULARIO MODAL -->
    <q-dialog v-model="modal" persistent transition-show="scale" transition-hide="scale">
      <q-card class="mod">
        <q-card-section class="modal-header">
          <div class="text-h6 text-weight-bold">
            {{ editando ? 'Editar Servicio' : 'Nuevo Servicio' }}
          </div>
          <q-btn icon="close" flat round dense v-close-popup @click="cerrar" />
        </q-card-section>

        <q-card-section class="q-pa-lg">
          <q-form ref="formRef" @submit="guardar">
            <div class="row q-col-gutter-md">
              <div class="col-12">
                <q-input
                  v-model="form.cliente"
                  label="Nombre del Cliente"
                  dark
                  filled
                  placeholder="Ingrese nombre del cliente"
                  :rules="[val => !!val || 'Campo requerido']"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-input
                  v-model="form.marca"
                  label="Marca del Equipo"
                  dark
                  filled
                  placeholder="Ej: Dell, Samsung"
                  :rules="[val => !!val || 'Campo requerido']"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-input
                  v-model="form.modelo"
                  label="Modelo del Equipo"
                  dark
                  filled
                  placeholder="Ej: G15, S21"
                  :rules="[val => !!val || 'Campo requerido']"
                />
              </div>

              <div class="col-12">
                <q-input
                  v-model="form.imagen"
                  label="URL de la Imagen (Opcional)"
                  dark
                  filled
                  hint="Deja en blanco para usar la imagen por defecto"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-select
                  v-model="form.reparacion"
                  label="Tipo de Reparación"
                  :options="reparaciones"
                  dark
                  filled
                  placeholder="Seleccionar"
                  :rules="[val => !!val || 'Seleccione opción']"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-select
                  v-model="form.tecnico"
                  label="Técnico Asignado"
                  :options="tecnicos"
                  dark
                  filled
                  placeholder="Seleccionar"
                  :rules="[val => !!val || 'Seleccione un técnico']"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-input
                  v-model="form.fecha"
                  type="date"
                  label="Fecha Recepción"
                  dark
                  filled
                  stack-label
                  :rules="[val => !!val || 'Seleccione fecha']"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-input
                  v-model="form.hora"
                  type="time"
                  label="Hora Recepción"
                  dark
                  filled
                  stack-label
                  :rules="[val => !!val || 'Seleccione hora']"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-input
                  v-model.number="form.precio"
                  type="number"
                  label="Precio Total"
                  prefix="$"
                  dark
                  filled
                  :rules="[val => val >= 0 || 'Monto inválido']"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-select
                  v-model="form.metodo"
                  label="Método de Pago"
                  :options="metodos"
                  dark
                  filled
                  placeholder="Seleccionar"
                  :rules="[val => !!val || 'Seleccione opción']"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-select
                  v-model="form.pago"
                  label="Estado del Pago"
                  :options="pagos"
                  dark
                  filled
                  placeholder="Seleccionar"
                  :rules="[val => !!val || 'Seleccione opción']"
                />
              </div>

              <div class="col-12 col-sm-6">
                <q-select
                  v-model="form.estado"
                  label="Estado del Equipo"
                  :options="estados"
                  dark
                  filled
                  placeholder="Seleccionar"
                  :rules="[val => !!val || 'Seleccione opción']"
                />
              </div>

              <div class="col-12">
                <q-input
                  v-model="form.obs"
                  type="textarea"
                  label="Observaciones Técnicas"
                  dark
                  filled
                  rows="2"
                  placeholder="Detalles adicionales del daño o estado..."
                />
              </div>
            </div>

            <div class="bot q-mt-lg flex justify-end gap-2">
              <q-btn label="Cancelar" flat color="grey" @click="cerrar" />
              <q-btn
                :label="editando ? 'Actualizar Servicio' : 'Guardar Servicio'"
                color="primary"
                class="btn-primary-glow"
                type="submit"
              />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const lista = useLocalStorage('servicios', [])

const modal = ref(false)
const editando = ref(false)
const id = ref(null)
const formRef = ref(null)

const form = ref({
  cliente: '',
  marca: '',
  modelo: '',
  imagen: '',
  reparacion: null,
  tecnico: null,
  fecha: '',
  hora: '',
  precio: null,
  metodo: null,
  pago: null,
  estado: null,
  calificacion: 0,
  obs: ''
})

const reparaciones = [
  'Cambio de pantalla',
  'Cambio de batería',
  'Cambio de pin de carga',
  'Liberación',
  'Mantenimiento de software',
  'Cambio de flex',
  'Otros'
]

const tecnicos = ['Don Efraín', 'Carlos', 'Andrés']
const metodos = ['Efectivo', 'Transferencia', 'Tarjeta']
const pagos = ['Pagado', 'Pendiente', 'Abono']
const estados = ['Recibido', 'En reparación', 'Listo para entregar', 'Entregado']

function obtenerImagenPorDefecto(tipo) {
  if (tipo === 'Cambio de pantalla') {
    return 'https://images.unsplash.com/photo-1588872657578-7efd1f1555ed?auto=format&fit=crop&w=500&q=80'
  }
  return 'https://images.unsplash.com/photo-1517336714731-489689fd1ca8?auto=format&fit=crop&w=500&q=80'
}

function limpiar() {
  const hoy = new Date()
  const fechaActual = hoy.toISOString().split('T')[0]
  const horaActual = hoy.toTimeString().slice(0, 5)

  form.value = {
    cliente: '',
    marca: '',
    modelo: '',
    imagen: '',
    reparacion: null,
    tecnico: null,
    fecha: fechaActual,
    hora: horaActual,
    precio: null,
    metodo: null,
    pago: null,
    estado: null,
    calificacion: 0,
    obs: ''
  }

  if (formRef.value) {
    formRef.value.resetValidation()
  }
}

function nuevo() {
  limpiar()
  editando.value = false
  id.value = null
  modal.value = true
}

function cerrar() {
  modal.value = false
  limpiar()
}

function guardar() {
  if (!editando.value) {
    lista.value.push({
      id: Date.now(),
      ...form.value
    })
  } else {
    const index = lista.value.findIndex(item => item.id === id.value)
    if (index !== -1) {
      lista.value[index] = {
        id: id.value,
        ...form.value
      }
    }
  }

  modal.value = false
  limpiar()
}

function editarServicio(servicio) {
  editando.value = true
  id.value = servicio.id
  form.value = { ...servicio }
  modal.value = true
}

function eliminar(servicioId) {
  if (confirm('¿Desea eliminar este registro?')) {
    lista.value = lista.value.filter(item => item.id !== servicioId)
  }
}

function contarReparaciones() {
  return lista.value.filter(item => item.estado === 'En reparación').length
}

function contarPendientes() {
  return lista.value.filter(item => item.pago === 'Pendiente').length
}

function precio(valor) {
  return Number(valor || 0).toLocaleString('es-CO')
}

function colorPago(estado) {
  if (estado === 'Pagado') return 'positive'
  if (estado === 'Pendiente') return 'negative'
  return 'warning'
}

function colorEquipo(estado) {
  if (estado === 'Recibido') return 'info'
  if (estado === 'En reparación') return 'warning'
  if (estado === 'Listo para entregar') return 'positive'
  return 'grey-7'
}

function icono(estado) {
  if (estado === 'Recibido') return 'inventory_2'
  if (estado === 'En reparación') return 'build'
  if (estado === 'Listo para entregar') return 'check_circle'
  return 'done_all'
}
</script>

<style>
:root {
  --bg-dark: #0f172a;
  --border-card: rgba(255, 255, 255, 0.12);
  --primary-cyan: #38bdf8;
  --primary-blue: #2563eb;
  --text-main: #f8fafc;
  --text-muted: #94a3b8;
  --glass-shadow: 0 20px 40px rgba(0, 0, 0, 0.35);
}

body {
  margin: 0;
  background-color: var(--bg-dark);
  color: var(--text-main);
  font-family: 'Inter', -apple-system, sans-serif;
}

.app {
  min-height: 100vh;
  background-image: 
    linear-gradient(rgba(15, 23, 42, 0.8), rgba(15, 23, 42, 0.8)), 
    url('https://th.bing.com/th/id/R.da5d82717dfe43ca80d89fb21542b7a5?rik=s4519dPgB9D3fQ&pid=ImgRaw&r=0');
  background-size: cover;
  background-position: center;
  background-attachment: fixed;
}

.cab {
  background: rgba(15, 23, 42, 0.85) !important;
  backdrop-filter: blur(12px);
  border-bottom: 1px solid var(--border-card);
}

.cab .q-toolbar {
  min-height: 72px;
  max-width: 1300px;
  margin: 0 auto;
}

.text-gradient {
  background: linear-gradient(135deg, #38bdf8 0%, #818cf8 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
}

.btn-primary-glow {
  background: linear-gradient(135deg, var(--primary-blue), #4f46e5) !important;
  box-shadow: 0 4px 20px rgba(37, 99, 235, 0.4);
  border-radius: 10px;
  font-weight: 600;
}

.cont {
  max-width: 1300px;
  margin: 0 auto;
  padding: 32px 24px 60px;
}

/* HEADER / PORTADA HERO */
.banner-img {
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.15);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
}

.header-overlay {
  background: radial-gradient(circle, rgba(15, 23, 42, 0.65) 0%, rgba(15, 23, 42, 0.88) 100%);
  backdrop-filter: blur(5px);
}

.hero-title-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0 16px;
}

.hero-title {
  margin: 0;
  font-size: clamp(2.2rem, 4vw, 3rem);
  font-weight: 900;
  letter-spacing: -0.5px;
  line-height: 1.1;
}

.text-gradient-hero {
  background: linear-gradient(135deg, #ffffff 10%, #38bdf8 50%, #a855f7 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 2px 10px rgba(56, 189, 248, 0.3));
}

.hero-divider {
  width: 60px;
  height: 4px;
  background: linear-gradient(90deg, #38bdf8, #818cf8);
  border-radius: 2px;
  margin: 12px 0 16px;
  box-shadow: 0 0 12px rgba(56, 189, 248, 0.8);
}

.sub-pill {
  display: inline-flex;
  align-items: center;
  padding: 8px 18px;
  background: rgba(255, 255, 255, 0.07);
  border: 1px solid rgba(255, 255, 255, 0.18);
  backdrop-filter: blur(10px);
  border-radius: 30px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.25);
}

.sub-text {
  color: #f1f5f9;
  font-size: clamp(0.9rem, 1.5vw, 1.05rem);
  font-weight: 500;
}

/* MÉTRICAS */
.metric-card {
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: center;
  padding: 20px;
  border-radius: 18px;
  background: rgba(15, 23, 42, 0.55) !important;
  backdrop-filter: blur(12px);
  border: 1px solid var(--border-card) !important;
}

.metric-card-img .metric-bg-img {
  position: absolute;
  top: 0;
  right: 0;
  width: 100%;
  height: 100%;
  opacity: 0.15;
}

.metric-content {
  display: flex;
  align-items: center;
  gap: 16px;
  z-index: 1;
}

.metric-icon {
  width: 52px;
  height: 52px;
  border-radius: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.bg-blue-soft { background: rgba(56, 189, 248, 0.12); }
.bg-orange-soft { background: rgba(245, 158, 11, 0.12); }
.bg-red-soft { background: rgba(239, 68, 68, 0.12); }

.metric-title {
  display: block;
  font-size: 0.85rem;
  color: var(--text-muted);
}

.metric-value {
  font-size: 1.75rem;
  font-weight: 700;
}

/* TARJETAS DE SERVICIOS */
.full-height {
  height: 100%;
}

.tar {
  background: rgba(15, 23, 42, 0.55) !important;
  border: 1px solid var(--border-card) !important;
  backdrop-filter: blur(16px);
  border-radius: 20px;
  transition: all 0.3s ease;
  box-shadow: var(--glass-shadow);
  color: var(--text-main);
}

.tar:hover {
  transform: translateY(-6px);
  border-color: rgba(56, 189, 248, 0.4) !important;
}

.border-pago-pendiente {
  border-left: 4px solid #ef4444 !important;
}

.border-pago-abono {
  border-left: 4px solid #f59e0b !important;
}

.cli {
  display: flex;
  align-items: center;
  gap: 14px;
}

.avatar-glow {
  background: linear-gradient(135deg, #38bdf8, #2563eb);
  box-shadow: 0 4px 14px rgba(56, 189, 248, 0.4);
}

.nom {
  font-size: 1.1rem;
  font-weight: 700;
}

.eq {
  color: var(--primary-cyan);
  font-size: 0.88rem;
  display: flex;
  align-items: center;
}

.divider-dark {
  background: var(--border-card) !important;
}

.info-grid p {
  margin: 8px 0;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 10px;
  color: var(--text-muted);
}

.info-grid p b {
  color: var(--text-main);
}

.info-grid .q-icon {
  color: var(--primary-cyan);
}

.badge-custom {
  padding: 6px 12px;
  border-radius: 8px;
  font-weight: 600;
}

.cal {
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid var(--border-card);
  border-radius: 12px;
  padding: 10px;
}

.obs {
  margin-top: 14px;
  background: rgba(15, 23, 42, 0.4);
  border-left: 3px solid var(--primary-cyan);
  padding: 10px 12px;
  border-radius: 4px 8px 8px 4px;
  font-size: 0.85rem;
}

.obs p {
  margin: 4px 0 0;
  color: var(--text-muted);
}

/* TARJETA FIJA Y ESTADO VACÍO CENTRADO */
.card-nuevo-registro {
  height: 100%;
  min-height: 280px;
  border: 2px dashed rgba(56, 189, 248, 0.4);
  background: rgba(15, 23, 42, 0.45);
  backdrop-filter: blur(12px);
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.card-nuevo-registro.empty-state-card {
  min-height: 320px;
  border: 2px dashed rgba(56, 189, 248, 0.6);
  background: rgba(15, 23, 42, 0.65);
}

.card-nuevo-registro:hover {
  border-color: #38bdf8;
  background: rgba(56, 189, 248, 0.12);
  transform: translateY(-6px);
  box-shadow: 0 16px 32px rgba(56, 189, 248, 0.25);
}

.btn-add-circle {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  background: linear-gradient(135deg, #38bdf8, #2563eb);
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  box-shadow: 0 6px 20px rgba(56, 189, 248, 0.4);
}

/* MODAL */
.mod {
  background: #1e293b !important;
  color: var(--text-main);
  border: 1px solid var(--border-card);
  border-radius: 20px;
  width: 600px;
  max-width: 95vw;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid var(--border-card);
}
</style>