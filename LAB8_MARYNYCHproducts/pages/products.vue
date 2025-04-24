<template>
  <div class="max-w-7xl mx-auto p-6">
    <h1 class="text-4xl font-extrabold text-gray-900 mb-8 text-center">Список продуктів</h1>

    <div class="mb-8 flex justify-center">
      <input
          v-model="search"
          type="text"
          class="w-full max-w-lg p-4 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all"
          placeholder="Пошук продуктів..."
      />
    </div>

    <div v-if="rawProducts.length > 0" class="overflow-x-auto rounded-lg shadow-lg bg-white">
      <table class="min-w-full table-auto">
        <thead>
        <tr>
          <th class="px-4 py-3 text-sm font-semibold text-left" @click="sortData('image')">Фото</th>
          <th class="px-4 py-3 text-sm font-semibold text-left" @click="sortData('title')">Назва</th>
          <th class="px-4 py-3 text-sm font-semibold text-left" @click="sortData('description')">Опис</th>
          <th class="px-4 py-3 text-sm font-semibold text-left" @click="sortData('price')">Ціна</th>
          <th class="px-4 py-3 text-sm font-semibold text-left" @click="sortData('rating')">Оцінка</th>
          <th class="px-4 py-3 text-sm font-semibold text-left" @click="sortData('category')">Категорія</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="product in sortedRows" :key="product.id" class="hover:bg-gray-100">
          <td class="px-4 py-3">
            <img :src="product.thumbnail || getFirstImage(product)" alt="Product Image" class="w-24 h-24 object-cover rounded-md" />
          </td>
          <td class="px-4 py-3">{{ product.title }}</td>
          <td class="px-4 py-3 max-w-xs truncate">{{ product.description }}</td>
          <td class="px-4 py-3">${{ product.price.toFixed(2) }}</td>
          <td class="px-4 py-3">
            <span :class="product.rating < 4.5 ? 'text-red-600' : 'text-green-600'">{{ product.rating }}</span>
          </td>
          <td class="px-4 py-3">{{ product.category }}</td>
        </tr>
        </tbody>
      </table>
    </div>

    <div v-if="sortedRows.length > 0" class="mt-6 flex justify-between items-center">
      <button @click="page = Math.max(page - 1, 1)" class="px-6 py-3 text-sm font-medium text-white bg-blue-600 rounded-lg hover:bg-blue-700 transition-all" :disabled="page === 1">
        Назад
      </button>
      <span class="text-sm text-gray-600">Сторінка {{ page }} з {{ totalPages }}</span>
      <button @click="page = Math.min(page + 1, totalPages)" class="px-6 py-3 text-sm font-medium text-white bg-blue-600 rounded-lg hover:bg-blue-700 transition-all" :disabled="page === totalPages">
        Вперед
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const rawProducts = ref([])  
const search = ref('')
const page = ref(1)
const pageCount = 5
const sort = ref({ column: 'title', direction: 'asc' })

const { data: productsData, pending, error } = await useFetch('https://dummyjson.com/products')

onMounted(async () => {
  if (pending.value) {
    await new Promise(resolve => {
      const check = () => !pending.value ? resolve() : setTimeout(check, 100)
      check()
    })
  }

  if (productsData.value?.products) {
    rawProducts.value = productsData.value.products
  }
})

const filteredRows = computed(() =>
    rawProducts.value.filter(p =>
        p.title.toLowerCase().includes(search.value.toLowerCase()) ||
        p.description.toLowerCase().includes(search.value.toLowerCase()) ||
        p.category.toLowerCase().includes(search.value.toLowerCase())
    )
)

const sortData = (column) => {
  if (sort.value.column === column) {
    sort.value.direction = sort.value.direction === 'asc' ? 'desc' : 'asc'
  } else {
    sort.value.column = column
    sort.value.direction = 'asc'
  }
}

const sortedRows = computed(() => {
  return filteredRows.value.slice().sort((a, b) => {
    const valueA = a[sort.value.column]
    const valueB = b[sort.value.column]

    if (valueA < valueB) return sort.value.direction === 'asc' ? -1 : 1
    if (valueA > valueB) return sort.value.direction === 'asc' ? 1 : -1
    return 0
  })
})

const totalPages = computed(() => Math.ceil(filteredRows.value.length / pageCount))

const paginatedRows = computed(() => {
  const start = (page.value - 1) * pageCount
  return sortedRows.value.slice(start, start + pageCount)
})

const getFirstImage = (product) => product.images?.[0] || ''

</script>

<style scoped>
button:disabled {
  background-color: #cccccc;
}
table {
  width: 100%;
  border-collapse: collapse;
}
th {
  cursor: pointer;
}
</style>
