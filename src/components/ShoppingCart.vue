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

      <!-- 👇 重新设计的结算区域 👇 -->
      <div class="bg-white p-6 md:p-8 rounded-3xl shadow-sm border border-gray-100 flex flex-col space-y-6">

        <!-- 🌟新增：VIP 专属折扣提示横幅 -->
        <div v-if="vipInfo.rate < 1" :class="['rounded-2xl p-4 flex items-center justify-between border', vipInfo.bg, vipInfo.border]">
          <div class="flex items-center space-x-3">
            <span class="text-2xl">✨</span>
            <div>
              <p :class="['text-sm font-black uppercase tracking-widest', vipInfo.color]">{{ vipInfo.name }} Privilege</p>
              <p :class="['text-xs font-bold mt-0.5 opacity-80', vipInfo.color]">Automatic {{ ((1 - vipInfo.rate) * 100).toFixed(0) }}% OFF applied</p>
            </div>
          </div>
          <span :class="['text-xl font-black', vipInfo.color]">- ¥{{ savedAmount.toFixed(2) }}</span>
        </div>

        <div class="flex items-end justify-between border-t border-gray-100 pt-6">
          <div>
            <p class="text-sm text-gray-500 font-medium mb-1">
              <span v-if="vipInfo.rate < 1" class="line-through text-gray-400 mr-2">¥{{ subtotal.toFixed(2) }}</span>
              Subtotal (Shipping: ¥10.00)
            </p>
            <div class="flex items-end space-x-2">
              <span class="text-3xl font-black text-gray-900">¥{{ finalTotal.toFixed(2) }}</span>
            </div>
          </div>
          <button @click="checkout" :disabled="isCheckingOut || cartItems.length === 0"
                  class="px-8 py-4 bg-gray-900 text-white rounded-2xl font-bold hover:bg-indigo-600 transition-all shadow-lg shadow-gray-200 disabled:bg-gray-300 disabled:shadow-none">
            {{ isCheckingOut ? 'Processing...' : 'Checkout Now' }}
          </button>
        </div>
      </div>
      <!-- 👆 结算区域修改结束 👆 -->

    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const cartItems = ref([]);
const isCheckingOut = ref(false);
const currentUser = ref({ id: 1, vip_level: 0 }); // 默认 fallback

// 🌟新增：获取本地缓存的用户信息，提取会员等级
const initUser = () => {
  const savedUser = localStorage.getItem('user');
  if (savedUser) {
    currentUser.value = JSON.parse(savedUser);
  }
};

// 获取购物车
const fetchCart = async () => {
  const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/cart/${currentUser.value.id}`);
  const result = await res.json();
  if (result.success) cartItems.value = result.data;
};

// 🌟新增：计算 VIP 对应的文案、折扣率和主题色
const vipInfo = computed(() => {
  const level = currentUser.value.vip_level || 0;
  if (level === 3) return { name: 'Diamond', rate: 0.85, color: 'text-indigo-600', bg: 'bg-indigo-50', border: 'border-indigo-100' };
  if (level === 2) return { name: 'Gold', rate: 0.90, color: 'text-amber-600', bg: 'bg-amber-50', border: 'border-amber-100' };
  if (level === 1) return { name: 'Silver', rate: 0.95, color: 'text-slate-600', bg: 'bg-slate-50', border: 'border-slate-200' };
  return { name: 'Standard', rate: 1.0, color: 'text-gray-600', bg: 'bg-transparent', border: 'border-transparent' };
});

const updateQuantity = async (cartId, newQuantity) => {
  if (newQuantity === 0 && !confirm('Are you sure you want to remove this book?')) return;

  try {
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/cart/${cartId}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ quantity: newQuantity })
    });
    if ((await res.json()).success) fetchCart();
  } catch (err) {
    console.error('Update failed');
  }
};

// 原始总价（商品数量 * 单价）
const subtotal = computed(() => {
  return cartItems.value.reduce((sum, item) => sum + (item.price * item.quantity), 0);
});

// 🌟新增：为你省下的金额
const savedAmount = computed(() => {
  return subtotal.value * (1 - vipInfo.value.rate);
});

// 🌟新增：打折后 + 运费的最终价格
const finalTotal = computed(() => {
  return (subtotal.value * vipInfo.value.rate) + 10;
});

// 结算
const checkout = async () => {
  isCheckingOut.value = true;
  try {
    const res = await fetch('https://bookstore-backend-60vr.onrender.com/api/checkout', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ consumer_id: currentUser.value.id })
    });

    const result = await res.json();
    if (result.success) {
      alert(`🎉 Order placed successfully! ${result.message || ''}`);
      fetchCart();
    } else {
      alert(`Checkout failed: ${result.message}`);
    }
  } catch (err) {
    alert('Checkout failed: Network error');
  } finally {
    isCheckingOut.value = false;
  }
};

onMounted(() => {
  initUser();
  fetchCart();
});
</script>