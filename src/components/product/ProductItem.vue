<template>
  <q-card class="product-card">
    <q-card-section class="q-pa-none">
      <img :src="imageSource" alt="Product Image" class="product-image" style="width: 40%" />
    </q-card-section>

    <q-card-section>
      <div class="text-h6">{{ title }}</div>
      <div class="text-subtitle2 text-secondary">{{ displayCategory }}</div>
    </q-card-section>

    <q-separator />

    <q-card-section>
      <div>{{ product.description ?? 'Sin descripción disponible' }}</div>
    </q-card-section>

    <q-card-section class="row items-center justify-between">
      <div class="text-subtitle1 text-primary">{{ formattedPrice }}</div>
      <q-chip v-if="product.stock != null" :label="`Stock: ${product.stock}`" dense outline />
    </q-card-section>
  </q-card>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  product: { type: Object, required: true },
})

const title = computed(() => props.product.name ?? props.product.title ?? 'Producto')
const imageSource = computed(
  () =>
    props.product.imageUrl ??
    props.product.ImageUrl ??
    props.product.image ??
    props.product.Image ??
    null,
)

const displayCategory = computed(() => {
  const category = props.product.category ?? props.product.Category
  if (category == null) {
    return 'Sin categoría'
  }
  if (typeof category === 'string') {
    return category
  }
  return (
    category.name ??
    category.description ??
    category.id ??
    category.Id ??
    (String(props.product.CategoryId ?? props.product.categoryId ?? '') || 'Sin categoría')
  )
})

const formattedPrice = computed(() => {
  const price = Number(props.product.price ?? props.product.unitPrice ?? props.product.Price ?? 0)
  return `$ ${Number.isNaN(price) ? 0 : price.toFixed(2)}`
})
</script>

<style scoped>
.product-card {
  min-height: 100%;
}
.product-image {
  object-fit: cover;
}
</style>
