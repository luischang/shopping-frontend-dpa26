<template>
  <div class="q-pa-md flex flex-center">
    <q-card class="q-pa-lg" flat bordered>
      <q-card-section>
        <div class="text-h6">Iniciar sesión</div>
      </q-card-section>

      <q-separator />

      <q-card-section>
        <q-form @submit.prevent="handleSubmit" class="column q-gutter-md">
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
            autocomplete="current-password"
          />

          <div class="text-negative" v-if="error">{{ error }}</div>

          <q-btn type="submit" label="Entrar" color="primary" :loading="loading" unelevated />
        </q-form>
      </q-card-section>
    </q-card>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useQuasar } from 'quasar'
import axios from 'axios'
import { useRouter } from 'vue-router'

const router = useRouter()

const BASE_URL = 'http://localhost'
const PORT = 5272
const api = axios.create({
  baseURL: `${BASE_URL}:${PORT}`,
  headers: {
    'Content-Type': 'application/json',
  },
})

const $q = useQuasar()
const loading = ref(false)
const error = ref('')
const form = reactive({
  email: '',
  password: '',
})
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

const handleSubmit = async () => {
  error.value = ''

  if (!form.email || !form.password) {
    error.value = 'Completa los campos email y contraseña.'
    return
  }

  if (!emailRegex.test(form.email)) {
    error.value = 'Ingresa un email válido.'
    return
  }

  loading.value = true

  try {
    const response = await api.post('/api/user/signin', {
      email: form.email,
      password: form.password,
    })

    $q.notify({
      type: 'positive',
      message: 'Inicio de sesión correcto',
    })
    //Add token in localStorage
    localStorage.setItem('token', response.data.token)

    router.push('/')
  } catch (err) {
    console.log('Login error:', err)
    if (axios.isAxiosError(err)) {
      error.value = err.response?.data?.message || 'Error en inicio de sesión.'
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
  max-width: 420px;
}
</style>
