<template>
  <q-page class="q-pa-md">
    <div class="row justify-center">
      <div class="column" style="max-width: 1200px; width: 100%">
        <div class="row items-center justify-between q-mb-md">
          <div>
            <div class="text-h5">Catálogo de productos</div>
            <div class="text-subtitle2 text-secondary">
              Usa los filtros para encontrar productos por nombre, categoría y precio.
            </div>
          </div>
        </div>

        <q-card flat bordered class="q-pa-md q-mb-md">
          <product-filter
            :categories="categories"
            :filters="filters"
            @update:filters="updateFilters"
          />
        </q-card>

        <product-list :products="products" :filters="filters" :loading="loading" :error="error" />
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import axios from 'axios'
import ProductFilter from '../components/product/ProductFilter.vue'
import ProductList from '../components/product/ProductList.vue'

const BASE_URL = 'http://localhost'
const PORT = 5272
const api = axios.create({
  baseURL: `${BASE_URL}:${PORT}`,
  headers: { 'Content-Type': 'application/json' },
})

const products = ref([])
const categories = ref([])
const loading = ref(true)
const error = ref('')

const filters = reactive({
  search: '',
  categoryId: '',
  minPrice: '',
  maxPrice: '',
})

const getAuthHeaders = () => {
  const token = localStorage.getItem('token')
  return token ? { Authorization: `Bearer ${token}` } : {}
}

const buildProductParams = () => {
  const params = {}
  if (filters.search) {
    params.Name = filters.search
    params.Search = filters.search
  }
  if (filters.categoryId) {
    params.CategoryId = filters.categoryId
  }
  if (filters.minPrice !== '' && !Number.isNaN(Number(filters.minPrice))) {
    params.MinPrice = Number(filters.minPrice)
  }
  if (filters.maxPrice !== '' && !Number.isNaN(Number(filters.maxPrice))) {
    params.MaxPrice = Number(filters.maxPrice)
  }
  return params
}

const loadCategories = async () => {
  const response = await api.get('/api/category', {
    headers: getAuthHeaders(),
  })
  categories.value = Array.isArray(response.data) ? response.data : []
}

const loadProducts = async () => {
  const response = await api.get('/api/product', {
    headers: getAuthHeaders(),
    params: buildProductParams(),
  })
  products.value = Array.isArray(response.data) ? response.data : []
}

const loadData = async () => {
  loading.value = true
  error.value = ''

  try {
    await Promise.all([loadCategories(), loadProducts()])
  } catch (err) {
    console.error('ProductPage error', err)
    error.value = 'No se pudieron cargar los productos. Intenta nuevamente.'
  } finally {
    loading.value = false
  }
}

const updateFilters = async (newFilters) => {
  filters.search = newFilters.search ?? ''
  filters.categoryId = newFilters.categoryId ?? ''
  filters.minPrice = newFilters.minPrice ?? ''
  filters.maxPrice = newFilters.maxPrice ?? ''

  loading.value = true
  error.value = ''

  try {
    await loadProducts()
  } catch (err) {
    console.error('ProductPage filter error', err)
    error.value = 'No se pudieron aplicar los filtros. Intenta nuevamente.'
  } finally {
    loading.value = false
  }
}

onMounted(loadData)
</script>
