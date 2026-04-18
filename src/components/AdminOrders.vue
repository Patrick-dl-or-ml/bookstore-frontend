<template>
  <div class="max-w-7xl mx-auto py-10 px-6 font-sans relative">

    <div class="absolute top-0 right-0 w-96 h-96 bg-indigo-50 rounded-full blur-[100px] -z-10"></div>

    <header class="mb-10 flex flex-col md:flex-row md:items-center justify-between gap-6">
      <div class="flex items-center space-x-4">
        <h1 class="text-3xl font-black text-gray-900 tracking-tight">Command Center</h1>
        <span class="px-3 py-1 bg-gray-900 text-white text-[10px] font-bold rounded-lg uppercase tracking-widest">
          {{ orders.length }} Orders
        </span>
      </div>

      <div class="relative w-full md:w-96 group">
        <input v-model="searchQuery" type="text" placeholder="Search Order ID..."
               class="w-full p-4 pl-12 bg-white/60 backdrop-blur-md border border-gray-200 rounded-2xl focus:ring-4 focus:ring-indigo-100/50 focus:border-indigo-400 transition-all font-bold text-gray-900 shadow-sm" />
        <svg class="absolute left-4 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>
      </div>
    </header>

    <main class="flex flex-col min-h-[600px]">
      <div v-if="paginatedOrders.length === 0" class="py-20 text-center bg-white/50 rounded-3xl border border-dashed border-gray-300">
        <p class="text-gray-400 font-bold">No orders found matching your criteria.</p>
      </div>

      <div class="space-y-4 flex-1">
        <div v-for="order in paginatedOrders" :key="order.sale_id"
             @click="openDetail(order)"
             class="group flex flex-col md:flex-row md:items-center justify-between p-6 bg-white/60 backdrop-blur-md border border-white hover:border-indigo-100 rounded-3xl shadow-sm hover:shadow-xl hover:-translate-y-0.5 transition-all duration-300 cursor-pointer">

          <div class="flex items-center space-x-6 mb-4 md:mb-0">
            <div class="h-16 w-16 rounded-2xl bg-gray-50 border border-gray-100 flex flex-col items-center justify-center shadow-inner flex-shrink-0">
              <span class="text-[10px] font-black text-gray-400 uppercase tracking-widest">ID</span>
              <span class="text-lg font-black text-indigo-600">{{ String(order.sale_id).padStart(5, '0') }}</span>
            </div>
            <div>
              <h3 class="text-lg font-black text-gray-900 group-hover:text-indigo-600 transition-colors">
                Total: ¥{{ order.total_price }}
              </h3>
              <p class="text-[11px] font-bold text-gray-400 tracking-wider">
                Placed on: {{ new Date(order.order_time).toLocaleString() }}
              </p>
            </div>
          </div>

          <div class="flex items-center space-x-8">
            <div class="flex flex-col items-end space-y-2">
              <span :class="statusColor(order.payment_status)" class="px-3 py-1 rounded-lg text-[10px] font-black uppercase tracking-widest shadow-sm border border-white/50">
                {{ order.payment_status || '未支付' }}
              </span>
              <span :class="statusColor(order.delivery_status)" class="px-3 py-1 rounded-lg text-[10px] font-black uppercase tracking-widest shadow-sm border border-white/50">
                {{ order.delivery_status || '未发货' }}
              </span>
            </div>

            <div class="w-10 h-10 flex items-center justify-center rounded-full bg-gray-50 text-gray-400 group-hover:bg-indigo-600 group-hover:text-white transition-all">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M9 5l7 7-7 7" /></svg>
            </div>
          </div>
        </div>
      </div>

      <div v-if="totalPages > 1" class="flex flex-wrap items-center justify-center gap-2 mt-12 pb-8">
        <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1" class="w-10 h-10 flex items-center justify-center rounded-full bg-white/60 hover:bg-white text-gray-600 font-black shadow-sm disabled:opacity-40 transition-all">←</button>
        <template v-for="(item, index) in visiblePages" :key="index">
          <span v-if="item === '...'" class="px-1 text-gray-400 font-black tracking-widest self-end pb-2">...</span>
          <button v-else @click="changePage(item)" :class="[currentPage === item ? 'bg-gray-900 text-white scale-110 shadow-lg' : 'bg-white/60 hover:bg-white text-gray-700']" class="w-10 h-10 rounded-full font-black text-sm flex items-center justify-center shadow-sm transition-all">{{ item }}</button>
        </template>
        <button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages" class="w-10 h-10 flex items-center justify-center rounded-full bg-white/60 hover:bg-white text-gray-600 font-black shadow-sm disabled:opacity-40 transition-all">→</button>
      </div>
    </main>

    <Transition name="modal">
      <div v-if="selectedOrder" class="fixed inset-0 z-[100] flex items-center justify-center p-4 sm:p-6">
        <div class="absolute inset-0 bg-gray-900/40 backdrop-blur-md" @click="selectedOrder = null"></div>

        <div class="relative bg-white rounded-[2.5rem] shadow-4xl w-full max-w-5xl overflow-hidden animate-zoomIn flex flex-col md:flex-row h-[700px]">

          <div class="w-full md:w-2/5 p-10 border-r border-gray-100 flex flex-col bg-gray-50/50">
            <div class="mb-8">
              <h3 class="text-2xl font-black text-gray-900 mb-1">Dispatch Control</h3>
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Order #{{ String(selectedOrder.sale_id).padStart(5, '0') }}</p>
            </div>

            <div class="flex-1 space-y-6">
              <div class="p-6 bg-white border border-gray-100 rounded-2xl shadow-sm">
                <p class="text-[10px] font-black text-indigo-500 uppercase tracking-widest mb-3">Shipping Destination (Snapshot)</p>
                <div class="text-sm font-bold text-gray-700 whitespace-pre-wrap leading-relaxed">
                  {{ selectedOrder.address_snapshot || 'No shipping address recorded.' }}
                </div>
              </div>

              <div class="grid grid-cols-2 gap-4">
                <div class="p-4 bg-white border border-gray-100 rounded-2xl shadow-sm">
                  <p class="text-[9px] font-black text-gray-400 uppercase tracking-widest mb-1">Total Due</p>
                  <p class="text-xl font-black text-gray-900">¥{{ selectedOrder.total_price }}</p>
                </div>
                <div class="p-4 bg-white border border-gray-100 rounded-2xl shadow-sm">
                  <p class="text-[9px] font-black text-gray-400 uppercase tracking-widest mb-1">Amount Paid</p>
                  <p class="text-xl font-black text-emerald-600">¥{{ selectedOrder.paid_amount || 0 }}</p>
                </div>
              </div>
            </div>

            <button @click="markAsShipped"
                    :disabled="selectedOrder.delivery_status !== '未发货'"
                    :class="selectedOrder.delivery_status === '未发货' ? 'bg-indigo-600 text-white shadow-lg hover:bg-indigo-700 hover:-translate-y-1' : 'bg-gray-200 text-gray-400 cursor-not-allowed'"
                    class="w-full py-5 rounded-xl font-black text-sm uppercase tracking-widest transition-all mt-4">
              {{ selectedOrder.delivery_status === '未发货' ? 'Mark as Shipped' : 'Already Processed' }}
            </button>
          </div>

          <div class="w-full md:w-3/5 p-10 flex flex-col bg-white">
            <div class="flex justify-between items-end mb-6">
              <h3 class="text-2xl font-black text-gray-900">Line Items</h3>
              <span class="text-xs font-bold text-gray-400">{{ orderDetails.length }} unique books</span>
            </div>

            <div class="flex-1 overflow-y-auto space-y-4 pr-2 custom-scrollbar">
              <div v-if="orderDetails.length === 0" class="py-10 text-center text-gray-400 font-bold text-sm">
                Loading details...
              </div>

              <div v-for="item in orderDetails" :key="item.detail_id" class="flex items-center p-4 border border-gray-100 rounded-2xl hover:bg-gray-50 transition-colors">
                <div class="h-16 w-12 bg-gray-100 rounded-lg overflow-hidden mr-4 shadow-sm">
                  <img :src="`https://images.unsplash.com/photo-1544947950-fa07a98d237f?auto=format&fit=crop&q=80&w=100&sig=${item.book_id}`" class="object-cover w-full h-full" />
                </div>
                <div class="flex-1">
                  <p class="font-black text-gray-900 leading-tight mb-1">{{ item.book_name }}</p>
                  <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest">Unit: ¥{{ item.unit_price }}</p>
                </div>
                <div class="text-right ml-4">
                  <p class="text-sm font-black text-gray-900">x{{ item.quantity }}</p>
                  <p class="text-sm font-black text-indigo-600 mt-1">¥{{ (item.quantity * item.unit_price).toFixed(2) }}</p>
                </div>
              </div>
            </div>
          </div>

          <button @click="selectedOrder = null" class="absolute top-6 right-6 w-10 h-10 flex items-center justify-center bg-gray-100 rounded-full text-gray-400 hover:text-gray-900 transition-all">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M6 18L18 6M6 6l12 12" /></svg>
          </button>
        </div>
      </div>
    </Transition>

  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';

const orders = ref([]);
const searchQuery = ref('');
const selectedOrder = ref(null);
const orderDetails = ref([]);

const currentPage = ref(1);
const limit = ref(25);

// 状态颜色徽章逻辑
const statusColor = (status) => {
  if (status === '已支付' || status === '已签收') return 'bg-emerald-100 text-emerald-700';
  if (status === '未发货' || status === '未支付') return 'bg-amber-100 text-amber-700';
  if (status === '已发货') return 'bg-blue-100 text-blue-700';
  return 'bg-gray-100 text-gray-600';
};

const filteredOrders = computed(() => {
  if (!searchQuery.value) return orders.value;
  const q = searchQuery.value.toLowerCase();
  return orders.value.filter(o => String(o.sale_id).includes(q));
});

watch(searchQuery, () => { currentPage.value = 1; });

const totalPages = computed(() => Math.ceil(filteredOrders.value.length / limit.value) || 1);
const paginatedOrders = computed(() => {
  const start = (currentPage.value - 1) * limit.value;
  return filteredOrders.value.slice(start, start + limit.value);
});

const visiblePages = computed(() => {
  const total = totalPages.value, current = currentPage.value, range = [];
  for (let i = Math.max(2, current - 1); i <= Math.min(total - 1, current + 1); i++) range.push(i);
  if (current - 1 > 2) range.unshift('...');
  if (current + 1 < total - 1) range.push('...');
  range.unshift(1);
  if (total > 1) range.push(total);
  return range;
});

const changePage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page;
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }
};

const fetchOrders = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/orders')).json();
    if (res.success) orders.value = res.data;
  } catch (err) { console.error('Failed to fetch orders'); }
};

const fetchOrderDetails = async (saleId) => {
  try {
    const res = await (await fetch(`https://bookstore-backend-60vr.onrender.com/api/admin/orders/${saleId}/details`)).json();
    if (res.success) orderDetails.value = res.data;
  } catch (err) { console.error('Failed to fetch details'); }
};

const openDetail = (order) => {
  selectedOrder.value = order;
  orderDetails.value = [];
  fetchOrderDetails(order.sale_id);
};

const markAsShipped = async () => {
  if (!confirm('Confirm dispatch? The customer will be notified.')) return;
  try {
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/admin/orders/${selectedOrder.value.sale_id}/dispatch`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ delivery_status: '已发货' })
    });
    if ((await res.json()).success) {
      alert('Order marked as shipped!');
      selectedOrder.value.delivery_status = '已发货';
      fetchOrders();
    }
  } catch (err) { alert('Dispatch failed'); }
};

onMounted(() => { fetchOrders(); });
</script>

<style scoped>
.custom-scrollbar::-webkit-scrollbar { width: 4px; }
.custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
.custom-scrollbar::-webkit-scrollbar-thumb { background: #E5E7EB; border-radius: 4px; }
.custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #D1D5DB; }

.modal-enter-active, .modal-leave-active { transition: all 0.4s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; transform: scale(0.95); }
@keyframes zoomIn {
  from { opacity: 0; transform: scale(0.98) translateY(10px); }
  to { opacity: 1; transform: scale(1) translateY(0); }
}
.animate-zoomIn { animation: zoomIn 0.3s ease-out; }
</style>