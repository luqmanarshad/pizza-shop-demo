<script setup>
import { ref, computed, watch } from 'vue'

const STORAGE_KEY = 'pizza-demo-v1'

const menu = [
  { id: 1, name: 'Chicken Tikka', cat: 'Pizza', emoji: 'ðŸ•', desc: 'Tikka chicken, onions, green peppers, mozzarella.', price: { S: 750, M: 1250, L: 1750 } },
  { id: 2, name: 'Pepperoni Feast', cat: 'Pizza', emoji: 'ðŸ•', desc: 'Double pepperoni with extra cheese.', price: { S: 800, M: 1350, L: 1900 } },
  { id: 3, name: 'Fajita Special', cat: 'Pizza', emoji: 'ðŸ•', desc: 'Fajita chicken, olives, capsicum, tangy sauce.', price: { S: 750, M: 1250, L: 1750 } },
  { id: 4, name: 'Veggie Garden', cat: 'Pizza', emoji: 'ðŸ¥¬', desc: 'Mushrooms, olives, corn, tomato, capsicum.', price: { S: 650, M: 1100, L: 1550 } },
  { id: 5, name: 'Crown Crust', cat: 'Pizza', emoji: 'ðŸ‘‘', desc: 'Cheese-stuffed crust with chicken and veggies.', price: { S: 900, M: 1500, L: 2100 } },
  { id: 6, name: 'Garlic Bread', cat: 'Sides', emoji: 'ðŸ¥–', desc: 'Toasted with garlic butter and cheese.', price: { S: 350, M: 350, L: 350 }, single: true },
  { id: 7, name: 'Crispy Wings (8 pcs)', cat: 'Sides', emoji: 'ðŸ—', desc: 'Hot and crispy with dip.', price: { S: 650, M: 650, L: 650 }, single: true },
  { id: 8, name: 'Cola 1.5L', cat: 'Drinks', emoji: 'ðŸ¥¤', desc: 'Chilled soft drink.', price: { S: 220, M: 220, L: 220 }, single: true },
  { id: 9, name: 'Mint Margarita', cat: 'Drinks', emoji: 'ðŸ¹', desc: 'Fresh mint, lemon and soda.', price: { S: 250, M: 250, L: 250 }, single: true },
]
const toppings = [{ name: 'Extra cheese', price: 150 }, { name: 'JalapeÃ±os', price: 80 }, { name: 'Mushrooms', price: 100 }]
const cats = ['All', 'Pizza', 'Sides', 'Drinks']
const sizeLabel = { S: 'Small', M: 'Medium', L: 'Large' }

function load() {
  try { const raw = localStorage.getItem(STORAGE_KEY); if (raw) return JSON.parse(raw) } catch (e) {}
  return { orders: [], counter: 1000 }
}
const store = ref(load())
watch(store, (v) => { try { localStorage.setItem(STORAGE_KEY, JSON.stringify(v)) } catch (e) {} }, { deep: true })

const view = ref('menu')
const cat = ref('All')
const visible = computed(() => menu.filter((m) => cat.value === 'All' || m.cat === cat.value))

// customizing item
const picking = ref(null)
const pick = ref({ size: 'M', tops: [], qty: 1 })
function openItem(item) { picking.value = item; pick.value = { size: item.single ? 'S' : 'M', tops: [], qty: 1 } }
const unitPrice = computed(() => {
  if (!picking.value) return 0
  const t = toppings.filter((x) => pick.value.tops.includes(x.name)).reduce((s, x) => s + x.price, 0)
  return picking.value.price[pick.value.size] + t
})

const cart = ref([])
function addToCart() {
  const item = picking.value
  cart.value.push({
    key: Date.now() + Math.random(),
    name: item.name,
    size: item.single ? null : pick.value.size,
    tops: [...pick.value.tops],
    qty: pick.value.qty,
    unit: unitPrice.value,
  })
  picking.value = null
}
const subtotal = computed(() => cart.value.reduce((s, i) => s + i.unit * i.qty, 0))
const delivery = computed(() => (checkout.value.type === 'Delivery' && subtotal.value > 0 ? 150 : 0))
const total = computed(() => subtotal.value + delivery.value)
const cartCount = computed(() => cart.value.reduce((s, i) => s + i.qty, 0))

const checkout = ref({ name: '', phone: '', address: '', type: 'Delivery' })
const placed = ref(null)
function placeOrder() {
  if (!cart.value.length) return
  store.value.counter += 1
  const order = {
    id: store.value.counter,
    customer: checkout.value.name,
    phone: checkout.value.phone,
    address: checkout.value.type === 'Delivery' ? checkout.value.address : '',
    type: checkout.value.type,
    items: cart.value.map(({ name, size, tops, qty }) => ({ name, size, tops, qty })),
    total: total.value,
    status: 'New',
    time: new Date().toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' }),
  }
  store.value.orders.unshift(order)
  placed.value = order
  cart.value = []
  checkout.value = { name: '', phone: '', address: '', type: 'Delivery' }
  view.value = 'done'
}

const flow = ['New', 'Preparing', 'Ready', 'Delivered']
const nextLabel = { New: 'Start preparing', Preparing: 'Mark ready', Ready: 'Mark delivered' }
function advance(o) { const i = flow.indexOf(o.status); if (i < flow.length - 1) o.status = flow[i + 1] }
const statusStyle = { New: 'bg-sky-100 text-sky-700', Preparing: 'bg-amber-100 text-amber-700', Ready: 'bg-emerald-100 text-emerald-700', Delivered: 'bg-slate-200 text-slate-600' }
const activeCount = computed(() => store.value.orders.filter((o) => o.status !== 'Delivered').length)
const salesToday = computed(() => store.value.orders.reduce((s, o) => s + o.total, 0))
const money = (n) => 'Rs. ' + n.toLocaleString('en-PK')
function clearOrders() { store.value = { orders: [], counter: 1000 } }
</script>

<template>
  <div class="bg-amber-100 px-4 py-2 text-center text-xs text-amber-900">
    Demo project with sample data (saved in your browser only) Â·
    <a href="https://luqmanarshad.github.io/" class="font-semibold underline">Back to portfolio</a>
  </div>

  <header class="border-b border-orange-200 bg-white">
    <div class="mx-auto flex max-w-6xl flex-wrap items-center justify-between gap-3 px-4 py-4">
      <div><h1 class="text-2xl font-extrabold text-orange-600">ðŸ• Slice House</h1><p class="text-xs text-slate-500">Pizza ordering &amp; kitchen system</p></div>
      <nav class="flex gap-1 rounded-xl bg-orange-50 p-1" aria-label="Views">
        <button class="rounded-lg px-4 py-2 text-sm font-semibold" :class="view === 'menu' || view === 'done' ? 'bg-orange-600 text-white' : 'text-slate-600'" @click="view = 'menu'">Menu</button>
        <button class="relative rounded-lg px-4 py-2 text-sm font-semibold" :class="view === 'cart' ? 'bg-orange-600 text-white' : 'text-slate-600'" @click="view = 'cart'">Cart
          <span v-if="cartCount" class="ml-1 rounded-full bg-white px-1.5 text-xs text-orange-600 ring-1 ring-orange-300">{{ cartCount }}</span></button>
        <button class="rounded-lg px-4 py-2 text-sm font-semibold" :class="view === 'kitchen' ? 'bg-orange-600 text-white' : 'text-slate-600'" @click="view = 'kitchen'">Kitchen
          <span v-if="activeCount" class="ml-1 rounded-full bg-orange-600 px-1.5 text-xs text-white">{{ activeCount }}</span></button>
      </nav>
    </div>
  </header>

  <main class="mx-auto max-w-6xl px-4 py-6">
    <!-- Menu -->
    <section v-if="view === 'menu'">
      <div class="flex flex-wrap gap-2">
        <button v-for="c in cats" :key="c" class="rounded-full border px-4 py-1.5 text-sm font-medium transition" :class="cat === c ? 'border-orange-600 bg-orange-600 text-white' : 'border-orange-200 bg-white text-slate-600 hover:bg-orange-100'" @click="cat = c">{{ c }}</button>
      </div>
      <div class="mt-5 grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
        <article v-for="m in visible" :key="m.id" class="flex flex-col rounded-2xl border border-orange-100 bg-white p-5 shadow-sm">
          <div class="text-4xl" aria-hidden="true">{{ m.emoji }}</div>
          <h2 class="mt-3 font-bold">{{ m.name }}</h2>
          <p class="mt-1 flex-1 text-sm text-slate-500">{{ m.desc }}</p>
          <div class="mt-4 flex items-center justify-between">
            <span class="font-semibold text-orange-600">{{ m.single ? money(m.price.S) : 'from ' + money(m.price.S) }}</span>
            <button class="rounded-lg bg-orange-600 px-4 py-2 text-sm font-semibold text-white hover:bg-orange-500" @click="openItem(m)">Add</button>
          </div>
        </article>
      </div>
    </section>

    <!-- Cart -->
    <section v-if="view === 'cart'" class="grid gap-5 lg:grid-cols-5">
      <div class="rounded-2xl bg-white p-5 shadow-sm lg:col-span-3">
        <h2 class="font-bold">Your cart</h2>
        <ul class="mt-3 divide-y divide-slate-100">
          <li v-for="i in cart" :key="i.key" class="flex items-start justify-between gap-3 py-3">
            <div><p class="font-semibold">{{ i.qty }} Ã— {{ i.name }}<span v-if="i.size" class="font-normal text-slate-500"> ({{ sizeLabel[i.size] }})</span></p>
              <p v-if="i.tops.length" class="text-xs text-slate-500">+ {{ i.tops.join(', ') }}</p></div>
            <div class="text-right"><p class="font-semibold">{{ money(i.unit * i.qty) }}</p>
              <button class="text-xs text-red-600 hover:underline" @click="cart = cart.filter((x) => x.key !== i.key)">Remove</button></div>
          </li>
          <li v-if="!cart.length" class="py-10 text-center text-slate-400">Your cart is empty. <button class="font-semibold text-orange-600 underline" @click="view = 'menu'">Browse the menu</button></li>
        </ul>
      </div>
      <form class="space-y-3 rounded-2xl bg-white p-5 shadow-sm lg:col-span-2" @submit.prevent="placeOrder">
        <h2 class="font-bold">Checkout</h2>
        <div class="flex gap-2"><label v-for="t in ['Delivery', 'Pickup']" :key="t" class="flex-1 cursor-pointer rounded-lg border px-3 py-2 text-center text-sm font-medium" :class="checkout.type === t ? 'border-orange-600 bg-orange-50 text-orange-700' : 'border-slate-200'">
          <input v-model="checkout.type" type="radio" :value="t" class="sr-only" />{{ t }}</label></div>
        <input v-model="checkout.name" required placeholder="Your name" aria-label="Your name" class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm" />
        <input v-model="checkout.phone" required type="tel" placeholder="Phone number" aria-label="Phone number" class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm" />
        <input v-if="checkout.type === 'Delivery'" v-model="checkout.address" required placeholder="Delivery address" aria-label="Delivery address" class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm" />
        <dl class="space-y-1 border-t border-slate-100 pt-3 text-sm">
          <div class="flex justify-between"><dt class="text-slate-500">Subtotal</dt><dd>{{ money(subtotal) }}</dd></div>
          <div class="flex justify-between"><dt class="text-slate-500">Delivery</dt><dd>{{ money(delivery) }}</dd></div>
          <div class="flex justify-between text-base font-bold"><dt>Total</dt><dd>{{ money(total) }}</dd></div>
        </dl>
        <button :disabled="!cart.length" class="w-full rounded-lg bg-orange-600 px-4 py-3 text-sm font-semibold text-white hover:bg-orange-500 disabled:cursor-not-allowed disabled:opacity-50">Place order</button>
      </form>
    </section>

    <!-- Confirmation -->
    <section v-if="view === 'done' && placed" class="mx-auto max-w-md rounded-2xl bg-white p-8 text-center shadow-sm">
      <div class="text-5xl">âœ…</div>
      <h2 class="mt-3 text-xl font-extrabold">Order #{{ placed.id }} placed!</h2>
      <p class="mt-1 text-sm text-slate-500">Thanks {{ placed.customer }}. Total {{ money(placed.total) }} Â· {{ placed.type }}</p>
      <div class="mt-5 flex justify-center gap-3">
        <button class="rounded-lg bg-orange-600 px-4 py-2 text-sm font-semibold text-white" @click="view = 'kitchen'">Open kitchen view</button>
        <button class="rounded-lg border border-slate-300 px-4 py-2 text-sm font-semibold" @click="view = 'menu'">Order more</button>
      </div>
    </section>

    <!-- Kitchen -->
    <section v-if="view === 'kitchen'">
      <div class="flex flex-wrap items-center justify-between gap-3">
        <div class="flex gap-4 text-sm"><span class="rounded-xl bg-white px-4 py-2 shadow-sm"><b>{{ store.orders.length }}</b> orders</span><span class="rounded-xl bg-white px-4 py-2 shadow-sm">Sales <b>{{ money(salesToday) }}</b></span></div>
        <button class="rounded-lg border border-slate-300 bg-white px-3 py-2 text-xs font-medium" @click="clearOrders">Clear orders</button>
      </div>
      <div class="mt-5 grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        <article v-for="o in store.orders" :key="o.id" class="rounded-2xl border border-orange-100 bg-white p-5 shadow-sm">
          <div class="flex items-center justify-between"><h3 class="font-bold">#{{ o.id }} Â· {{ o.customer }}</h3><span class="rounded-full px-2.5 py-0.5 text-xs font-semibold" :class="statusStyle[o.status]">{{ o.status }}</span></div>
          <p class="text-xs text-slate-500">{{ o.time }} Â· {{ o.type }}<span v-if="o.address"> Â· {{ o.address }}</span></p>
          <ul class="mt-3 space-y-1 text-sm"><li v-for="(it, idx) in o.items" :key="idx">{{ it.qty }} Ã— {{ it.name }}<span v-if="it.size"> ({{ sizeLabel[it.size] }})</span><span v-if="it.tops.length" class="text-xs text-slate-500"> + {{ it.tops.join(', ') }}</span></li></ul>
          <div class="mt-4 flex items-center justify-between"><span class="font-semibold">{{ money(o.total) }}</span>
            <button v-if="o.status !== 'Delivered'" class="rounded-lg bg-slate-900 px-3 py-1.5 text-xs font-semibold text-white hover:bg-slate-700" @click="advance(o)">{{ nextLabel[o.status] }}</button></div>
        </article>
        <p v-if="!store.orders.length" class="col-span-full py-16 text-center text-slate-400">No orders yet. Add something from the menu and place an order.</p>
      </div>
    </section>
  </main>

  <!-- Item dialog -->
  <div v-if="picking" class="fixed inset-0 z-50 flex items-end justify-center bg-black/50 p-4 sm:items-center" role="dialog" aria-modal="true" :aria-label="'Customize ' + picking.name" @click.self="picking = null">
    <div class="w-full max-w-md rounded-2xl bg-white p-6 shadow-xl">
      <h2 class="text-lg font-extrabold">{{ picking.emoji }} {{ picking.name }}</h2>
      <p class="text-sm text-slate-500">{{ picking.desc }}</p>
      <div v-if="!picking.single" class="mt-4">
        <p class="text-xs font-semibold uppercase text-slate-500">Size</p>
        <div class="mt-2 flex gap-2"><button v-for="(label, k) in sizeLabel" :key="k" type="button" class="flex-1 rounded-lg border px-2 py-2 text-sm font-medium" :class="pick.size === k ? 'border-orange-600 bg-orange-50 text-orange-700' : 'border-slate-200'" @click="pick.size = k">{{ label }}<br /><span class="text-xs">{{ money(picking.price[k]) }}</span></button></div>
        <template v-if="picking.cat === 'Pizza'">
          <p class="mt-4 text-xs font-semibold uppercase text-slate-500">Extra toppings</p>
          <label v-for="t in toppings" :key="t.name" class="mt-2 flex cursor-pointer items-center justify-between rounded-lg border border-slate-200 px-3 py-2 text-sm"><span><input v-model="pick.tops" type="checkbox" :value="t.name" class="mr-2" />{{ t.name }}</span><span class="text-slate-500">+{{ money(t.price) }}</span></label>
        </template>
      </div>
      <div class="mt-5 flex items-center justify-between">
        <div class="flex items-center gap-3"><button type="button" class="h-8 w-8 rounded-full border border-slate-300" aria-label="Decrease quantity" @click="pick.qty = Math.max(1, pick.qty - 1)">âˆ’</button><span class="w-5 text-center font-semibold">{{ pick.qty }}</span><button type="button" class="h-8 w-8 rounded-full border border-slate-300" aria-label="Increase quantity" @click="pick.qty++">+</button></div>
        <button class="rounded-lg bg-orange-600 px-5 py-2.5 text-sm font-semibold text-white hover:bg-orange-500" @click="addToCart">Add Â· {{ money(unitPrice * pick.qty) }}</button>
      </div>
    </div>
  </div>
</template>
