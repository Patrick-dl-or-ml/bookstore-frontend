<template>
  <div class="max-w-7xl mx-auto py-10 px-6 font-sans relative">

    <div class="absolute top-0 right-0 w-96 h-96 bg-indigo-50 rounded-full blur-[100px] -z-10"></div>

    <header class="mb-10 flex flex-col md:flex-row md:items-center justify-between gap-6">
      <div class="flex items-center space-x-4">
        <h1 class="text-3xl font-black text-gray-900 tracking-tight">Client Relations</h1>
        <span class="px-3 py-1 bg-gray-900 text-white text-[10px] font-bold rounded-lg uppercase tracking-widest">
          {{ consumers.length }} Registered
        </span>
      </div>

      <div class="relative w-full md:w-96 group">
        <input v-model="searchQuery" type="text" placeholder="Search by name, email or phone..."
               class="w-full p-4 pl-12 bg-white/60 backdrop-blur-md border border-gray-200 rounded-2xl focus:ring-4 focus:ring-indigo-100/50 focus:border-indigo-400 transition-all font-bold text-gray-900 shadow-sm" />
        <svg class="absolute left-4 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>
      </div>
    </header>

    <main class="flex flex-col min-h-[600px]">
      <div v-if="paginatedConsumers.length === 0" class="py-20 text-center bg-white/50 rounded-3xl border border-dashed border-gray-300">
        <p class="text-gray-400 font-bold">No clients found matching your query.</p>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 flex-1">
        <div v-for="user in paginatedConsumers" :key="user.consumer_id"
             @click="openDetail(user)"
             class="group p-6 bg-white/60 backdrop-blur-md border border-white hover:border-indigo-100 rounded-3xl shadow-sm hover:shadow-xl hover:-translate-y-1 transition-all duration-300 cursor-pointer flex flex-col h-fit">

          <div class="flex items-start justify-between mb-4">
            <div class="flex items-center space-x-4">
              <div class="w-14 h-14 rounded-full bg-gradient-to-br from-indigo-100 to-purple-100 flex items-center justify-center text-indigo-600 font-black text-xl shadow-inner border border-white">
                {{ user.consumer_name ? user.consumer_name.charAt(0).toUpperCase() : 'U' }}
              </div>
              <div>
                <h3 class="text-lg font-black text-gray-900 group-hover:text-indigo-600 transition-colors">{{ user.consumer_name || 'Unnamed User' }}</h3>
                <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest mt-0.5">ID: {{ String(user.consumer_id).padStart(4, '0') }}</p>
              </div>
            </div>
            <span :class="vipColor(user.vip_level)" class="px-3 py-1 rounded-lg text-[10px] font-black uppercase tracking-widest shadow-sm border border-white/50">
              VIP {{ user.vip_level }}
            </span>
          </div>

          <div class="mt-auto pt-4 border-t border-gray-100/50 flex justify-between items-end">
            <div>
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">Contact</p>
              <p class="text-sm font-bold text-gray-600">{{ user.phone || user.email || 'No contact provided' }}</p>
            </div>
            <div class="text-right">
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">Integral</p>
              <p class="text-lg font-black text-indigo-600">{{ user.integral }}</p>
            </div>
          </div>
        </div>
      </div>

      <div v-if="totalPages > 1" class="flex flex-wrap items-center justify-center gap-2 mt-12 pb-8">
        <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1" class="w-10 h-10 flex items-center justify-center rounded-full bg-white/60 hover:bg-white text-gray-600 font-black shadow-sm disabled:opacity-40 disabled:hover:bg-white/60 transition-all">
          ←
        </button>

        <template v-for="(item, index) in visiblePages" :key="index">
          <span v-if="item === '...'" class="px-1 text-gray-400 font-black tracking-widest self-end pb-2">...</span>
          <button v-else @click="changePage(item)"
                  :class="[currentPage === item ? 'bg-gray-900 text-white scale-110 shadow-lg' : 'bg-white/60 hover:bg-white text-gray-700']"
                  class="w-10 h-10 rounded-full font-black text-sm flex items-center justify-center shadow-sm transition-all">
            {{ item }}
          </button>
        </template>

        <button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages" class="w-10 h-10 flex items-center justify-center rounded-full bg-white/60 hover:bg-white text-gray-600 font-black shadow-sm disabled:opacity-40 disabled:hover:bg-white/60 transition-all">
          →
        </button>
      </div>
    </main>

    <Transition name="modal">
      <div v-if="selectedUser" class="fixed inset-0 z-[100] flex items-center justify-center p-4 sm:p-6">
        <div class="absolute inset-0 bg-gray-900/40 backdrop-blur-md" @click="selectedUser = null"></div>

        <div class="relative bg-white rounded-[2.5rem] shadow-4xl w-full max-w-4xl overflow-hidden animate-zoomIn flex h-[600px]">

          <div class="w-1/2 p-10 border-r border-gray-100 flex flex-col bg-gray-50/50">
            <h3 class="text-2xl font-black text-gray-900 mb-8">Profile Editor</h3>

            <div class="space-y-5 flex-1">
              <div>
                <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Client Name</label>
                <input v-model="editForm.consumer_name" class="w-full p-4 bg-white border border-gray-100 rounded-xl font-bold text-gray-900 focus:ring-2 focus:ring-indigo-500 outline-none" />
              </div>
              <div>
                <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Email Address</label>
                <input v-model="editForm.email" class="w-full p-4 bg-white border border-gray-100 rounded-xl font-bold text-gray-900 outline-none" />
              </div>
              <div class="grid grid-cols-2 gap-4">
                <div>
                  <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">VIP Level</label>
                  <input v-model="editForm.vip_level" type="number" class="w-full p-4 bg-white border border-gray-100 rounded-xl font-bold text-gray-900 outline-none" />
                </div>
                <div>
                  <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Integral Points</label>
                  <input v-model="editForm.integral" type="number" step="0.01" class="w-full p-4 bg-white border border-gray-100 rounded-xl font-bold text-indigo-600 outline-none" />
                </div>
              </div>
            </div>

            <button @click="handleUpdate" class="w-full py-4 bg-indigo-600 text-white rounded-xl font-black text-sm uppercase tracking-widest shadow-lg hover:bg-indigo-700 transition-all active:scale-95">
              Save Profile
            </button>
          </div>

          <div class="w-1/2 p-10 flex flex-col bg-white">
            <div class="flex justify-between items-end mb-6">
              <h3 class="text-2xl font-black text-gray-900">Address Book</h3>
              <span class="text-xs font-bold text-gray-400">{{ userAddresses.length }} records</span>
            </div>

            <div class="flex-1 overflow-y-auto space-y-4 pr-2 custom-scrollbar">
              <div v-if="userAddresses.length === 0" class="py-10 text-center text-gray-400 font-bold text-sm">
                No addresses saved by this client.
              </div>

              <div v-for="addr in userAddresses" :key="addr.address_id" class="p-5 border border-gray-100 rounded-2xl bg-gray-50/50 relative">
                <div v-if="addr.is_default" class="absolute top-4 right-4 text-[9px] font-black text-indigo-500 bg-indigo-50 px-2 py-1 rounded uppercase tracking-widest">Default</div>
                <p class="font-black text-gray-900 text-lg mb-1">{{ addr.receiver_name }}</p>
                <p class="text-xs font-bold text-gray-500 mb-3">{{ addr.receiver_phone }}</p>
                <p class="text-sm font-medium text-gray-700 leading-relaxed">
                  {{ addr.province }} {{ addr.city }} {{ addr.district }}<br>
                  {{ addr.detail_address }}
                </p>
              </div>
            </div>
          </div>

          <button @click="selectedUser = null" class="absolute top-6 right-6 w-10 h-10 flex items-center justify-center bg-gray-100 rounded-full text-gray-400 hover:text-gray-900 transition-all">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M6 18L18 6M6 6l12 12" /></svg>
          </button>
        </div>
      </div>
    </Transition>

  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';

const consumers = ref([]);
const searchQuery = ref('');
const selectedUser = ref(null);
const userAddresses = ref([]);
const editForm = ref({});

// 🌟 分页核心状态
const currentPage = ref(1);
const limit = ref(39); //
// VIP 颜色逻辑
const vipColor = (level) => {
  if (level >= 5) return 'bg-yellow-100 text-yellow-700';
  if (level >= 2) return 'bg-gray-200 text-gray-700';
  return 'bg-indigo-50 text-indigo-600';
};

const filteredConsumers = computed(() => {
  if (!searchQuery.value) return consumers.value;
  const q = searchQuery.value.toLowerCase();
  return consumers.value.filter(c =>
      (c.consumer_name && c.consumer_name.toLowerCase().includes(q)) ||
      (c.email && c.email.toLowerCase().includes(q)) ||
      (c.phone && c.phone.includes(q))
  );
});

watch(searchQuery, () => {
  currentPage.value = 1;
});

const totalPages = computed(() => {
  return Math.ceil(filteredConsumers.value.length / limit.value) || 1;
});

const paginatedConsumers = computed(() => {
  const start = (currentPage.value - 1) * limit.value;
  const end = start + limit.value;
  return filteredConsumers.value.slice(start, end);
});

const visiblePages = computed(() => {
  const total = totalPages.value;
  const current = currentPage.value;
  const range = [];

  for (let i = Math.max(2, current - 1); i <= Math.min(total - 1, current + 1); i++) {
    range.push(i);
  }
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

const fetchConsumers = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/consumers')).json();
    if (res.success) consumers.value = res.data;
  } catch (err) { console.error('Failed to fetch consumers'); }
};

const fetchUserAddresses = async (consumerId) => {
  try {
    const res = await (await fetch(`https://bookstore-backend-60vr.onrender.com/api/admin/consumers/${consumerId}/addresses`)).json();
    if (res.success) userAddresses.value = res.data;
  } catch (err) { console.error('Failed to fetch addresses'); }
};

const openDetail = (user) => {
  selectedUser.value = user;
  editForm.value = { ...user };
  userAddresses.value = [];
  fetchUserAddresses(user.consumer_id);
};

const handleUpdate = async () => {
  try {
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/admin/consumers/${selectedUser.value.consumer_id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(editForm.value)
    });
    if ((await res.json()).success) {
      alert('Profile updated successfully');
      selectedUser.value = null;
      fetchConsumers();
    }
  } catch (err) { alert('Update failed'); }
};

onMounted(() => { fetchConsumers(); });
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