<template>
  <div class="max-w-4xl mx-auto py-12 px-4 font-sans">
    <header class="mb-10">
      <h1 class="text-4xl font-black text-gray-900 tracking-tighter">My Orders</h1>
      <p class="text-sm font-bold text-gray-400 uppercase tracking-widest mt-2">Track your vault acquisitions</p>
    </header>

    <div v-if="isLoading" class="flex justify-center py-20">
      <div class="w-10 h-10 border-4 border-indigo-200 border-t-indigo-600 rounded-full animate-spin"></div>
    </div>

    <div v-else-if="orders.length === 0" class="bg-white p-20 text-center rounded-[2.5rem] border border-dashed border-gray-200">
      <p class="text-gray-400 font-bold text-lg">You haven't placed any orders yet.</p>
      <router-link to="/" class="text-indigo-600 font-black uppercase text-xs tracking-widest mt-4 inline-block hover:underline">Go to Store</router-link>
    </div>

    <div v-else class="space-y-8">
      <div v-for="order in orders" :key="order.sale_id" class="bg-white rounded-[2.5rem] shadow-sm border border-gray-100 overflow-hidden hover:shadow-md transition-shadow">
        <!-- 订单头部信息 -->
        <div class="bg-gray-50 px-8 py-4 flex justify-between items-center border-bottom border-gray-100">
          <div class="flex space-x-6">
            <div>
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Order ID</p>
              <p class="text-sm font-bold text-gray-900">#{{ order.sale_id }}</p>
            </div>
            <div>
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Date</p>
              <p class="text-sm font-bold text-gray-900">{{ formatDate(order.created_at) }}</p>
            </div>
          </div>
          <div class="text-right">
            <span class="px-3 py-1 bg-emerald-100 text-emerald-600 text-[10px] font-black rounded-lg uppercase tracking-widest">
              {{ order.payment_status }}
            </span>
          </div>
        </div>

        <!-- 订单商品明细 -->
        <div class="p-8">
          <div class="space-y-4">
            <div v-for="item in order.items" :key="item.book_id" class="flex items-center justify-between">
              <div class="flex items-center space-x-4">
                <div class="w-10 h-14 bg-gray-100 rounded-md overflow-hidden flex-shrink-0">
                  <img :src="`https://images.unsplash.com/photo-1544947950-fa07a98d237f?auto=format&fit=crop&q=80&w=100&sig=${item.book_id}`" class="w-full h-full object-cover" />
                </div>
                <div>
                  <p class="font-bold text-gray-900">{{ item.book_name }}</p>
                  <p class="text-xs text-gray-400">Qty: {{ item.quantity }} × ¥{{ item.unit_price }}</p>
                </div>
              </div>
              <p class="font-black text-gray-900">¥{{ (item.quantity * item.unit_price).toFixed(2) }}</p>
            </div>
          </div>

          <div class="mt-8 pt-6 border-t border-gray-50 flex justify-between items-end">
            <div>
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Status</p>
              <p class="text-sm font-bold text-indigo-600">{{ order.delivery_status }}</p>
            </div>
            <div class="text-right">
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Total Amount</p>
              <p class="text-2xl font-black text-gray-900">¥{{ Number(order.total_price).toFixed(2) }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const orders = ref([]);
const isLoading = ref(true);

const formatDate = (dateStr) => {
  return new Date(dateStr).toLocaleDateString('zh-CN', { year: 'numeric', month: 'long', day: 'numeric' });
};

const fetchOrders = async () => {
  try {
    const savedUser = localStorage.getItem('user');
    const userId = savedUser ? JSON.parse(savedUser).id : 1;
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/orders/user/${userId}`);
    const result = await res.json();
    if (result.success) orders.value = result.data;
  } catch (err) {
    console.error('Failed to fetch orders');
  } finally {
    isLoading.value = false;
  }
};

onMounted(fetchOrders);
</script>