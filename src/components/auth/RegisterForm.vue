<template>
  <div class="q-pa-md flex flex-center">
    <q-card class="q-pa-lg" flat bordered>
      <q-card-section>
        <div class="text-h6">Registro de usuario</div>
      </q-card-section>

      <q-separator />

      <q-card-section>
        <q-form @submit.prevent="handleSubmit" class="column q-gutter-md">
          <q-input
            v-model="form.firstName"
            label="Nombre"
            clearable
            :rules="[(val) => !!val || 'El nombre es obligatorio']"
          />

          <q-input
            v-model="form.lastName"
            label="Apellido"
            clearable
            :rules="[(val) => !!val || 'El apellido es obligatorio']"
          />

          <q-input v-model="form.dateOfBirth" label="Fecha de nacimiento" type="date" clearable />

          <q-input v-model="form.country" label="País" clearable />

          <q-input v-model="form.address" label="Dirección" clearable />

          <q-input
            v-model="form.email"
            label="Email"
            type="email"
            clearable
            :rules="[
              (val) => !!val || 'El email es obligatorio',
              (val) => emailRegex.test(val) || 'Email inválido',
            ]"
            autocomplete="email"
          />

          <q-input
            v-model="form.password"
            label="Password"
            type="password"
            clearable
            :rules="[(val) => !!val || 'La contraseña es obligatoria']"
            autocomplete="new-password"
          />

          <div class="text-negative" v-if="error">{{ error }}</div>

          <q-btn type="submit" label="Registrarse" color="primary" :loading="loading" unelevated />
        </q-form>
      </q-card-section>
    </q-card>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useQuasar } from 'quasar'
import axios from 'axios'

const BASE_URL = 'http://localhost'
const PORT = 5272
const api = axios.create({
  baseURL: `${BASE_URL}:${PORT}`,
  headers: {
    'Content-Type': 'application/json',
  },
})

const $q = useQuasar()
const router = useRouter()
const loading = ref(false)
const error = ref('')
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

const form = reactive({
  firstName: '',
  lastName: '',
  dateOfBirth: '',
  country: '',
  address: '',
  email: '',
  password: '',
})

const handleSubmit = async () => {
  error.value = ''

  if (!form.firstName || !form.lastName || !form.email || !form.password) {
    error.value = 'Completa los campos obligatorios.'
    return
  }

  if (!emailRegex.test(form.email)) {
    error.value = 'Ingresa un email válido.'
    return
  }

  loading.value = true

  try {
    await api.post('/api/user/signup', {
      firstName: form.firstName,
      lastName: form.lastName,
      dateOfBirth: form.dateOfBirth || null,
      country: form.country || null,
      address: form.address || null,
      email: form.email,
      password: form.password,
    })

    $q.notify({
      type: 'positive',
      message: 'Registro exitoso. Redirigiendo al login...',
    })

    router.push('/login')
  } catch (err) {
    if (axios.isAxiosError(err)) {
      error.value = err.response?.data?.message || 'Error al registrar el usuario.'
    } else {
      error.value = 'Error de red, inténtalo de nuevo.'
    }
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.q-card {
  min-width: 320px;
  max-width: 520px;
}
</style>
