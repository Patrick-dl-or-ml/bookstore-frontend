<template>
  <div class="max-w-4xl mx-auto py-12 px-4 sm:px-6 lg:px-8 font-sans">
    <h1 class="text-3xl font-black text-gray-900 mb-8">My Cart</h1>

    <div v-if="cartItems.length === 0" class="bg-white p-16 text-center rounded-3xl border border-gray-100 shadow-sm">
      <div class="text-6xl mb-4">🛒</div>
      <h3 class="text-xl font-bold text-gray-900 mb-2">Your cart is empty</h3>
      <p class="text-gray-500 font-medium">Looks like you haven't added any books yet.</p>
    </div>

    <div v-else class="space-y-6">
      <div class="bg-white rounded-3xl shadow-sm border border-gray-100 overflow-hidden">
        <ul class="divide-y divide-gray-100">
          <li v-for="item in cartItems" :key="item.cart_id" class="p-6 flex items-center justify-between hover:bg-gray-50 transition-colors">

            <div class="flex items-center space-x-6 flex-1">
              <div class="h-24 w-16 bg-gray-200 rounded-lg overflow-hidden flex-shrink-0 shadow-sm">
                <img :src="`https://images.unsplash.com/photo-1544947950-fa07a98d237f?auto=format&fit=crop&q=80&w=200&sig=${item.book_id}`" class="h-full w-full object-cover"/>
              </div>
              <div>
                <h3 class="text-lg font-bold text-gray-900">{{ item.book_name }}</h3>
                <p class="text-sm text-gray-500 font-medium">{{ item.author }}</p>
                <p class="text-lg font-black text-gray-900 mt-2">¥{{ item.price }}</p>
              </div>
            </div>

            <div class="flex items-center space-x-4 bg-gray-100 rounded-xl p-1 shadow-inner">
              <button @click="updateQuantity(item.cart_id, item.quantity - 1)"
                      class="w-8 h-8 flex items-center justify-center bg-white rounded-lg text-gray-600 font-bold hover:shadow hover:text-indigo-600 transition-all">
                -
              </button>
              <span class="w-8 text-center font-black text-gray-900">{{ item.quantity }}</span>
              <button @click="updateQuantity(item.cart_id, item.quantity + 1)"
                      class="w-8 h-8 flex items-center justify-center bg-white rounded-lg text-gray-600 font-bold hover:shadow hover:text-indigo-600 transition-all">
                +
              </button>
            </div>

          </li>
        </ul>
      </div>

      <div class="bg-white p-6 rounded-3xl shadow-sm border border-gray-100 flex items-center justify-between">
        <div>
          <p class="text-sm text-gray-500 font-medium mb-1">Subtotal (Shipping: ¥10.00)</p>
          <p class="text-3xl font-black text-gray-900">¥{{ (subtotal + 10).toFixed(2) }}</p>
        </div>
        <button @click="checkout" :disabled="isCheckingOut"
                class="px-8 py-4 bg-gray-900 text-white rounded-2xl font-bold hover:bg-indigo-600 transition-all shadow-lg shadow-gray-200 disabled:bg-gray-300">
          {{ isCheckingOut ? 'Processing...' : 'Checkout Now' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const cartItems = ref([]);
const isCheckingOut = ref(false);

const getCurrentUserId = () => {
  const savedUser = localStorage.getItem('user');
  return savedUser ? JSON.parse(savedUser).id : 1;
};

// 获取购物车
const fetchCart = async () => {
  const userId = getCurrentUserId();
  const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/cart/${userId}`);
  const result = await res.json();
  if (result.success) cartItems.value = result.data;
};

// 🌟 核心升级：修改数量
const updateQuantity = async (cartId, newQuantity) => {
  // 如果减到 0 提示确认
  if (newQuantity === 0 && !confirm('Are you sure you want to remove this book?')) return;

  try {
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/cart/${cartId}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ quantity: newQuantity })
    });
    if ((await res.json()).success) fetchCart(); // 刷新数据
  } catch (err) {
    console.error('Update failed');
  }
};

const subtotal = computed(() => {
  return cartItems.value.reduce((sum, item) => sum + (item.price * item.quantity), 0);
});

// 结算
const checkout = async () => {
  isCheckingOut.value = true;
  try {
    const res = await fetch('https://bookstore-backend-60vr.onrender.com/api/checkout', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ consumer_id: getCurrentUserId() })
    });
    if ((await res.json()).success) {
      alert('🎉 Order placed successfully!');
      fetchCart();
    }
  } catch (err) {
    alert('Checkout failed');
  } finally {
    isCheckingOut.value = false;
  }
};

onMounted(fetchCart);
</script>