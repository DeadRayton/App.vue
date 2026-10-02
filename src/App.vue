<script setup lang="ts">
import { ref, onMounted } from 'vue'
import Saludo from './components/Saludo.vue'

// Endpoint de la API
const API_URL = 'https://jsonplaceholder.typicode.com/posts'

// Estado reactivo
const publicaciones = ref<any[]>([])
const cargando = ref(true)
const modoEdicion = ref(false)

// Formulario para crear o editar
const formulario = ref({
  id: null as number | null,
  title: '',
  body: ''
})

// 1. GET: Listar publicaciones
const obtenerPublicaciones = async () => {
  cargando.value = true
  try {
    const respuesta = await fetch(`${API_URL}?_limit=8`)
    const datos = await respuesta.json()
    publicaciones.value = datos
  } catch (error) {
    console.error('Error al consultar la API:', error)
  } finally {
    cargando.value = false
  }
}

// 2. POST: Crear nueva publicación
const crearPublicacion = async () => {
  if (!formulario.value.title.trim() || !formulario.value.body.trim()) return

  try {
    const respuesta = await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json; charset=UTF-8' },
      body: JSON.stringify({
        title: formulario.value.title,
        body: formulario.value.body,
        userId: 1,
      }),
    })
    const nuevaPublicacion = await respuesta.json()
    publicaciones.value.unshift(nuevaPublicacion)
    limpiarFormulario()
  } catch (error) {
    console.error('Error al crear publicación:', error)
  }
}

// Cargar datos al formulario para editar
const prepararEdicion = (pub: any) => {
  modoEdicion.value = true
  formulario.value = {
    id: pub.id,
    title: pub.title,
    body: pub.body
  }
  // Desplaza suavemente hacia arriba donde está el formulario
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// 3. PUT: Actualizar publicación existente
const actualizarPublicacion = async () => {
  if (!formulario.value.id) return

  try {
    const respuesta = await fetch(`${API_URL}/${formulario.value.id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json; charset=UTF-8' },
      body: JSON.stringify({
        id: formulario.value.id,
        title: formulario.value.title,
        body: formulario.value.body,
        userId: 1,
      }),
    })
    const datosActualizados = await respuesta.json()

    // Buscamos la publicación en la lista y la actualizamos
    const index = publicaciones.value.findIndex(p => p.id === formulario.value.id)
    if (index !== -1) {
      publicaciones.value[index] = datosActualizados
    }
    limpiarFormulario()
  } catch (error) {
    console.error('Error al actualizar publicación:', error)
  }
}

// 4. DELETE: Eliminar publicación
const eliminarPublicacion = async (id: number) => {
  const confirmar = confirm(`¿Estás seguro de que deseas eliminar la publicación #${id}?`)
  if (!confirmar) return

  try {
    await fetch(`${API_URL}/${id}`, {
      method: 'DELETE'
    })
    // Removemos la publicación del arreglo reactivo
    publicaciones.value = publicaciones.value.filter(pub => pub.id !== id)
  } catch (error) {
    console.error('Error al eliminar publicación:', error)
  }
}

// Controla si se envía como POST o como PUT
const manejarEnvio = () => {
  if (modoEdicion.value) {
    actualizarPublicacion()
  } else {
    crearPublicacion()
  }
}

// Cancela o limpia el formulario
const limpiarFormulario = () => {
  formulario.value = { id: null, title: '', body: '' }
  modoEdicion.value = false
}

onMounted(() => {
  obtenerPublicaciones()
})
</script>

<template>
  <main class="contenedor">
    <Saludo />

    <!-- SECCIÓN FORMULARIO (POST / PUT) -->
    <section class="card-formulario">
      <h2>{{ modoEdicion ? `✏️ Editar Publicación #${formulario.id} (PUT)` : '➕ Nueva Publicación (POST)' }}</h2>
      <form @submit.prevent="manejarEnvio">
        <div class="campo">
          <label for="titulo">Título:</label>
          <input
            id="titulo"
            v-model="formulario.title"
            type="text"
            placeholder="Escribe el título aquí..."
            required
          />
        </div>

        <div class="campo">
          <label for="cuerpo">Contenido:</label>
          <textarea
            id="cuerpo"
            v-model="formulario.body"
            rows="3"
            placeholder="Escribe el texto de la publicación..."
            required
          ></textarea>
        </div>

        <div class="acciones-form">
          <button type="submit" class="btn btn-principal">
            {{ modoEdicion ? 'Actualizar Cambios (PUT)' : 'Guardar Publicación (POST)' }}
          </button>
          <button
            v-if="modoEdicion"
            type="button"
            class="btn btn-cancelar"
            @click="limpiarFormulario"
          >
            Cancelar Edición
          </button>
        </div>
      </form>
    </section>

    <hr class="separador" />

    <!-- SECCIÓN LISTADO (GET / DELETE) -->
    <div class="encabezado-lista">
      <h2>Listado de Publicaciones (GET)</h2>
      <span class="contador">Total: {{ publicaciones.length }}</span>
    </div>

    <p v-if="cargando" class="aviso">Cargando publicaciones desde la API...</p>

    <div v-else class="lista">
      <article v-for="pub in publicaciones" :key="pub.id" class="tarjeta">
        <div class="tarjeta-info">
          <span class="id-post">Post #{{ pub.id }}</span>
          <h3>{{ pub.title }}</h3>
          <p>{{ pub.body }}</p>
        </div>
        <div class="acciones-card">
          <button class="btn btn-editar" @click="prepararEdicion(pub)">
            Editar (PUT)
          </button>
          <button class="btn btn-eliminar" @click="eliminarPublicacion(pub.id)">
            Eliminar (DELETE)
          </button>
        </div>
      </article>
    </div>
  </main>
</template>

<style scoped>
.contenedor {
  max-width: 800px;
  margin: 30px auto;
  padding: 0 15px;
  font-family: system-ui, -apple-system, sans-serif;
  color: #2c3e50;
}

.separador {
  margin: 30px 0;
  border: 0;
  border-top: 1px solid #e2e8f0;
}

/* Formulario */
.card-formulario {
  background-color: #f8fafc;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
}

.card-formulario h2 {
  margin-top: 0;
  font-size: 1.2rem;
  color: #1e293b;
}

.campo {
  display: flex;
  flex-direction: column;
  margin-bottom: 12px;
}

.campo label {
  font-weight: 600;
  font-size: 0.9rem;
  margin-bottom: 4px;
}

.campo input,
.campo textarea {
  padding: 8px 12px;
  border: 1px solid #cbd5e1;
  border-radius: 4px;
  font-size: 0.95rem;
  font-family: inherit;
}

.acciones-form {
  display: flex;
  gap: 10px;
}

/* Botones */
.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  font-weight: 600;
  cursor: pointer;
  font-size: 0.88rem;
  transition: opacity 0.2s ease;
}

.btn:hover {
  opacity: 0.88;
}

.btn-principal {
  background-color: #10b981;
  color: white;
}

.btn-cancelar {
  background-color: #64748b;
  color: white;
}

.btn-editar {
  background-color: #3b82f6;
  color: white;
}

.btn-eliminar {
  background-color: #ef4444;
  color: white;
}

/* Lista de posts */
.encabezado-lista {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.encabezado-lista h2 {
  margin: 0;
  color: #38bdf8;
}

.contador {
  background-color: #0284c7;
  color: white;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 0.85rem;
  font-weight: 600;
}

.aviso {
  text-align: center;
  font-weight: 500;
  color: #64748b;
}

.lista {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.tarjeta {
  background-color: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 16px 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 12px;
}

.id-post {
  display: inline-block;
  font-size: 0.75rem;
  font-weight: bold;
  background-color: #e2e8f0;
  color: #475569;
  padding: 2px 8px;
  border-radius: 4px;
  margin-bottom: 8px;
}

.tarjeta h3 {
  margin: 0 0 8px 0;
  font-size: 1.1rem;
  text-transform: capitalize;
}

.tarjeta p {
  margin: 0;
  color: #4b5563;
  line-height: 1.4;
}

.acciones-card {
  display: flex;
  gap: 10px;
  margin-top: 5px;
}
</style>