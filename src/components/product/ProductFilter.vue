<template>
  <q-form class="row wrap q-gutter-md" @submit.prevent>
    <q-input
      filled
      v-model="localFilters.search"
      label="Buscar por nombre"
      debounce="300"
      clearable
      class="col-12 col-sm-6 col-md-3"
    />

    <q-select
      filled
      v-model="localFilters.categoryId"
      :options="categoryOptions"
      label="Categoría"
      clearable
      class="col-12 col-sm-6 col-md-3"
    />

    <q-input
      filled
      v-model.number="localFilters.minPrice"
      label="Precio mínimo"
      type="number"
      min="0"
      clearable
      class="col-12 col-sm-6 col-md-3"
    />

    <q-input
      filled
      v-model.number="localFilters.maxPrice"
      label="Precio máximo"
      type="number"
      min="0"
      clearable
      class="col-12 col-sm-6 col-md-3"
    />

    <div class="row items-center justify-end col-12">
      <q-btn label="Limpiar" flat color="primary" @click="resetFilters" />
    </div>
  </q-form>
</template>

<script setup>
import { computed, reactive, watch } from 'vue'

const props = defineProps({
  categories: { type: Array, default: () => [] },
  filters: {
    type: Object,
    default: () => ({
      search: '',
      categoryId: '',
      minPrice: '',
      maxPrice: '',
    }),
  },
})

const emit = defineEmits(['update:filters'])

const localFilters = reactive({
  search: props.filters.search ?? '',
  categoryId: props.filters.categoryId ?? '',
  minPrice: props.filters.minPrice ?? '',
  maxPrice: props.filters.maxPrice ?? '',
})

watch(
  () => props.filters,
  (next) => {
    localFilters.search = next.search ?? ''
    localFilters.categoryId = next.categoryId ?? ''
    localFilters.minPrice = next.minPrice ?? ''
    localFilters.maxPrice = next.maxPrice ?? ''
  },
  { deep: true, immediate: true },
)

watch(
  localFilters,
  (next) => {
    emit('update:filters', { ...next })
  },
  { deep: true },
)

const categoryOptions = computed(() =>
  props.categories.map((category) => ({
    label: category.name ?? category.description ?? String(category),
    value: category.id ?? category.categoryId ?? category,
  })),
)

const resetFilters = () => {
  localFilters.search = ''
  localFilters.categoryId = ''
  localFilters.minPrice = ''
  localFilters.maxPrice = ''
}
</script>
