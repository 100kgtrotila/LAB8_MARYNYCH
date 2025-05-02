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

    <div v-if="isLoading" class="flex justify-center items-center h-64">
      <div class="text-xl">Завантаження продуктів...</div>
    </div>

    <div v-else-if="errorMessage" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded">
      <p>Помилка: {{ errorMessage }}</p>
      <p>Спробуйте оновити сторінку</p>
    </div>

    <div v-else-if="rawProducts.length > 0" class="overflow-x-auto rounded-lg shadow-lg bg-white">
      <table class="min-w-full table-auto">
        <thead>
        <tr class="bg-gray-50">
          <th
              v-for="(column, index) in columns"
              :key="index"
              class="px-4 py-3 text-sm font-semibold text-left cursor-pointer"
              @click="sortData(column.value)"
          >
            {{ column.label }}
            <span v-if="sort.column === column.value" class="ml-1">
                {{ sort.direction === 'asc' ? '↑' : '↓' }}
              </span>
            <span v-else class="ml-1 opacity-30">↕</span>
          </th>
        </tr>
        </thead>
        <tbody>

        <tr v-for="product in paginatedRows" :key="product.id" class="hover:bg-gray-100 border-t">
          <td class="px-4 py-3">
            <img
                :src="product.thumbnail || getFirstImage(product)"
                :alt="product.title"
                class="w-24 h-24 object-cover rounded-md"
            />
          </td>
          <td class="px-4 py-3 font-medium">{{ product.title }}</td>
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


    <div v-else class="text-center py-8 bg-gray-50 rounded-lg">
      Немає продуктів для відображення
    </div>

>
    <div v-if="paginatedRows.length > 0" class="mt-6 flex justify-between items-center">
      <button
          @click="page = Math.max(page - 1, 1)"
          class="px-6 py-3 text-sm font-medium text-white bg-blue-600 rounded-lg hover:bg-blue-700 transition-all flex items-center"
          :disabled="page === 1"
      >

        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" viewBox="0 0 20 20" fill="currentColor">
          <path fill-rule="evenodd" d="M12.707 5.293a1 1 0 010 1.414L9.414 10l3.293 3.293a1 1 0 01-1.414 1.414l-4-4a1 1 0 010-1.414l4-4a1 1 0 011.414 0z" clip-rule="evenodd" />
        </svg>
        Назад
      </button>
      <span class="text-sm text-gray-600">
        Сторінка {{ page }} з {{ totalPages }} (Всього продуктів: {{ filteredRows.length }})
      </span>
      <button
          @click="page = Math.min(page + 1, totalPages)"
          class="px-6 py-3 text-sm font-medium text-white bg-blue-600 rounded-lg hover:bg-blue-700 transition-all flex items-center"
          :disabled="page === totalPages"
      >
        Вперед

        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 ml-1" viewBox="0 0 20 20" fill="currentColor">
          <path fill-rule="evenodd" d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z" clip-rule="evenodd" />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const rawProducts = ref([])
const search = ref('')
const page = ref(1)
const pageCount = 5
const sort = ref({ column: 'title', direction: 'asc' })
const isLoading = ref(true)
const errorMessage = ref(null)

const columns = [
  { label: 'Фото', value: 'thumbnail' },
  { label: 'Назва', value: 'title' },
  { label: 'Опис', value: 'description' },
  { label: 'Ціна', value: 'price' },
  { label: 'Оцінка', value: 'rating' },
  { label: 'Категорія', value: 'category' }
]

onMounted(async () => {
  try {
    isLoading.value = true
    const response = await fetch('https://dummyjson.com/products')

    if (!response.ok) {
      throw new Error(`HTTP помилка! Статус: ${response.status}`)
    }

    const data = await response.json()
    rawProducts.value = data.products || []
  } catch (err) {
    console.error('Помилка завантаження продуктів:', err)
    errorMessage.value = err.message || 'Помилка завантаження даних'
  } finally {
    isLoading.value = false
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
  return [...filteredRows.value].sort((a, b) => {
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

watch(search, () => {
  page.value = 1
})
</script>

<style scoped>
button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}
table {
  width: 100%;
  border-collapse: collapse;
}
th {
  cursor: pointer;
}
</style>