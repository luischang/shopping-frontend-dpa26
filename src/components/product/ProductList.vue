<template>
  <div>
    <div v-if="loading" class="row justify-center q-pa-md">
      <q-spinner color="primary" size="3rem" />
    </div>

    <div v-else-if="error" class="text-negative q-pa-md">
      {{ error }}
    </div>

    <div v-else-if="filteredProducts.length === 0" class="text-caption q-pa-md">
      No se encontraron productos con los filtros seleccionados.
    </div>

    <div v-else class="product-grid">
      <div
        v-for="product in filteredProducts"
        :key="product.id ?? product.productId ?? product.name"
        class="product-grid__item"
      >
        <product-item :product="product" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import ProductItem from './ProductItem.vue'

const props = defineProps({
  products: { type: Array, default: () => [] },
  filters: {
    type: Object,
    default: () => ({
      search: '',
      categoryId: '',
      minPrice: '',
      maxPrice: '',
    }),
  },
  loading: { type: Boolean, default: false },
  error: { type: String, default: '' },
})

const filteredProducts = computed(() => {
  const search = String(props.filters.search ?? '')
    .trim()
    .toLowerCase()
  const categoryId = String(props.filters.categoryId ?? '').trim()
  const minPrice = Number(props.filters.minPrice)
  const maxPrice = Number(props.filters.maxPrice)

  return props.products.filter((product) => {
    const name = String(product.name ?? product.title ?? '').toLowerCase()
    const description = String(product.description ?? '').toLowerCase()
    const productCategoryId = String(
      product.CategoryId ??
        product.categoryId ??
        product.category?.id ??
        product.Category?.Id ??
        product.category ??
        product.Category ??
        '',
    )
    const price = Number(product.price ?? product.unitPrice ?? product.Price ?? 0)

    if (search && !`${name} ${description}`.includes(search)) {
      return false
    }

    if (categoryId && productCategoryId !== categoryId) {
      return false
    }

    if (!Number.isNaN(minPrice) && minPrice > 0 && price < minPrice) {
      return false
    }

    if (!Number.isNaN(maxPrice) && maxPrice > 0 && price > maxPrice) {
      return false
    }

    return true
  })
})
</script>

<style scoped>
.product-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1rem;
}
.product-grid__item {
  display: flex;
}
.product-grid__item > * {
  width: 100%;
}
</style>
