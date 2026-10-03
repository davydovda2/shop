<script setup lang="ts">
import { ref, watch } from 'vue'

type Product = { id: number; name: string; price: number; category: string; image: string }

const products: Product[] = [
  { id: 1, name: 'Ноутбук Pro 14', price: 89990, category: 'Электроника', image: '💻' },
  { id: 2, name: 'Механическая клавиатура', price: 7500, category: 'Электроника', image: '⌨️' },
  { id: 3, name: 'Кружка разработчика', price: 690, category: 'Аксессуары', image: '☕' },
  { id: 4, name: 'Худи "Vue Master"', price: 3200, category: 'Одежда', image: '👕' },
  { id: 5, name: 'Мышь беспроводная', price: 2400, category: 'Электроника', image: '🖱️' },
  { id: 6, name: 'Стикерпак с логотипом', price: 150, category: 'Аксессуары', image: '🏷️' },
  { id: 7, name: 'Монитор 27"', price: 24990, category: 'Электроника', image: '🖥️' },
  { id: 8, name: 'Кепка "Frontend"', price: 1100, category: 'Одежда', image: '🧢' },
]

const categories = ['Все', 'Электроника', 'Аксессуары', 'Одежда']

const search = ref('')
const category = ref('Все')
const sort = ref('')
const promo = ref('')
const error = ref('')
const cart = ref<(Product & { qty: number })[]>([])

const saved = localStorage.getItem('cart')
if (saved) cart.value = JSON.parse(saved)
if (localStorage.getItem('promo') === 'WEB') promo.value = 'WEB'

watch(cart, (value) => localStorage.setItem('cart', JSON.stringify(value)), { deep: true })
watch(promo, (value) => localStorage.setItem('promo', value))

function discount() {
  return promo.value === 'WEB' ? 0.9 : 1
}

function price(value: number) {
  return Math.round(value * discount())
}

function money(value: number) {
  return value.toLocaleString('ru-RU') + ' ₽'
}

function filtered() {
  const list = []
  const query = search.value.toLowerCase()

  for (const product of products) {
    const matchName = product.name.toLowerCase().includes(query)
    const matchCategory = category.value === 'Все' || product.category === category.value
    if (matchName && matchCategory) list.push(product)
  }

  if (sort.value === 'up') list.sort((a, b) => a.price - b.price)
  if (sort.value === 'down') list.sort((a, b) => b.price - a.price)

  return list
}

function cartCount() {
  let count = 0
  for (const item of cart.value) count = count + item.qty
  return count
}

function cartTotal() {
  let sum = 0
  for (const item of cart.value) sum = sum + price(item.price) * item.qty
  return sum
}

function add(product: Product) {
  for (const item of cart.value) {
    if (item.id === product.id) {
      item.qty = item.qty + 1
      return
    }
  }
  cart.value.push({ ...product, qty: 1 })
}

function remove(id: number) {
  const rest = []
  for (const item of cart.value) {
    if (item.id !== id) rest.push(item)
  }
  cart.value = rest
}

function clearCart() {
  cart.value = []
}

function applyPromo() {
  if (promo.value.trim().toUpperCase() === 'WEB') {
    promo.value = 'WEB'
    error.value = ''
  } else {
    error.value = 'Неверный промокод'
  }
}
</script>

<template>
  <div class="app">
    <h1>Интернет-магазин</h1>

    <div class="filters">
      <input v-model="search" placeholder="Поиск" />
      <select v-model="category">
        <option v-for="item in categories" :key="item">{{ item }}</option>
      </select>
      <select v-model="sort">
        <option value="">Без сортировки</option>
        <option value="up">Сначала дешёвые</option>
        <option value="down">Сначала дорогие</option>
      </select>
    </div>

    <p v-if="filtered().length === 0">Ничего не найдено</p>

    <div v-for="product in filtered()" :key="product.id" class="card">
      <span class="card-name">{{ product.image }} {{ product.name }}</span>
      <span class="card-price">
        {{ money(price(product.price)) }}
        <s v-if="discount() < 1">{{ money(product.price) }}</s>
      </span>
      <button @click="add(product)">В корзину</button>
    </div>

    <div class="cart">
      <h2>Корзина ({{ cartCount() }})</h2>

      <p v-if="cart.length === 0">Корзина пуста</p>

      <div v-for="item in cart" :key="item.id" class="cart-item">
        <span>{{ item.image }} {{ item.name }} × {{ item.qty }}</span>
        <span>{{ money(price(item.price) * item.qty) }}</span>
        <button class="remove" @click="remove(item.id)">✕</button>
      </div>

      <div class="promo">
        <input v-model="promo" placeholder="Промокод" />
        <button @click="applyPromo">Применить</button>
      </div>

      <p v-if="error" class="error">{{ error }}</p>
      <p v-if="discount() < 1">Скидка 10%</p>

      <p class="total">Итого: {{ money(cartTotal()) }}</p>

      <button v-if="cart.length > 0" @click="clearCart">Очистить корзину</button>
    </div>
  </div>
</template>

<style scoped>
.app {
  max-width: 700px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  font-size: 24px;
}

h2 {
  font-size: 18px;
}

input,
select,
button {
  padding: 7px;
  border: 1px solid #bbb;
  border-radius: 4px;
}

button {
  background: #eee;
  cursor: pointer;
}

button:hover {
  background: #ddd;
}

.filters,
.card,
.cart-item,
.promo {
  display: flex;
  gap: 8px;
}

.filters {
  margin-bottom: 15px;
}

.filters input,
.card-name,
.cart-item span:first-child {
  flex: 1;
}

.card {
  align-items: center;
  margin-bottom: 8px;
  padding: 10px;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.card-price s {
  margin-left: 5px;
  color: #999;
  font-size: 12px;
}

.cart {
  margin-top: 20px;
  padding: 15px;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.cart-item {
  align-items: center;
  padding: 8px 0;
  border-bottom: 1px solid #eee;
}

.remove {
  border: none;
  background: none;
  color: #c0392b;
}

.promo {
  margin: 15px 0;
}

.error {
  color: #c0392b;
}

.total {
  font-size: 20px;
  font-weight: bold;
}
</style>