<template>
  <div class="max-w-7xl mx-auto py-10 px-6 font-sans relative">

    <header class="mb-10 flex flex-col md:flex-row md:items-center justify-between gap-6">
      <div class="flex items-center space-x-4">
        <h1 class="text-3xl font-black text-gray-900 tracking-tight">User Directory</h1>
        <span class="px-3 py-1 bg-indigo-600 text-white text-[10px] font-bold rounded-lg uppercase tracking-widest">
          {{ users.length }} Active Members
        </span>
      </div>
    </header>

    <main class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] shadow-sm overflow-hidden">
      <table class="w-full text-left border-collapse">
        <thead>
        <tr class="border-b border-gray-100">
          <th class="p-6 text-[10px] font-black text-gray-400 uppercase tracking-widest">User Details</th>
          <th class="p-6 text-[10px] font-black text-gray-400 uppercase tracking-widest text-center">Status & Tier</th>
          <th class="p-6 text-[10px] font-black text-gray-400 uppercase tracking-widest text-right">Finances</th>
          <th class="p-6 text-[10px] font-black text-gray-400 uppercase tracking-widest text-right">Actions</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="user in users" :key="user.consumer_id" class="border-b border-gray-50 hover:bg-gray-50/50 transition-colors group">
          <!-- 用户基础信息 & 注册时间 (对标 3.2.1) -->
          <td class="p-6">
            <div class="flex items-center space-x-4">
              <div class="h-12 w-12 rounded-2xl bg-gradient-to-br from-gray-100 to-gray-200 flex items-center justify-center font-black text-gray-400">
                {{ user.consumer_name.charAt(0).toUpperCase() }}
              </div>
              <div>
                <p class="font-black text-gray-900">{{ user.consumer_name }}</p>
                <p class="text-[10px] font-bold text-gray-400 tracking-wider">Joined: {{ formatTime(user.register_time) }}</p>
              </div>
            </div>
          </td>

          <!-- 等级 & 积分 -->
          <td class="p-6 text-center">
            <div class="flex flex-col items-center space-y-1">
                <span :class="getVipClass(user.vip_level)" class="px-3 py-1 rounded-lg text-[9px] font-black uppercase tracking-widest">
                  {{ getVipName(user.vip_level) }}
                </span>
              <span class="text-[9px] font-bold text-gray-400 uppercase tracking-widest">{{ user.integral }} Points</span>
            </div>
          </td>

          <!-- 余额展示与修改 (对标 3.2.1) -->
          <td class="p-6 text-right">
            <div class="flex flex-col items-end">
              <p class="text-sm font-black text-gray-900">¥{{ Number(user.balance || 0).toFixed(2) }}</p>
              <button @click="openEditBalance(user)" class="text-[9px] font-black text-indigo-600 uppercase tracking-widest hover:underline mt-1">Manage Balance</button>
            </div>
          </td>

          <!-- 删除操作 (带 3.2.1 安全检查) -->
          <td class="p-6 text-right">
            <button @click="deleteUser(user.consumer_id)" class="p-2 text-gray-300 hover:text-red-600 transition-colors">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>
            </button>
          </td>
        </tr>
        </tbody>
      </table>
    </main>

    <!-- 🌟 余额/等级编辑弹窗 (Modal) -->
    <div v-if="editUser" class="fixed inset-0 z-[100] flex items-center justify-center p-6">
      <div class="absolute inset-0 bg-gray-900/40 backdrop-blur-md" @click="editUser = null"></div>
      <div class="relative bg-white rounded-[2rem] p-8 w-full max-w-md shadow-2xl animate-zoomIn">
        <h3 class="text-xl font-black text-gray-900 mb-6">Manage User Finance</h3>

        <div class="space-y-4">
          <div>
            <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest block mb-2">Account Balance (¥)</label>
            <input v-model="editUser.balance" type="number" class="w-full p-4 bg-gray-50 border border-gray-100 rounded-xl font-black text-gray-900 focus:ring-2 focus:ring-indigo-400 outline-none" />
          </div>
          <div>
            <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest block mb-2">VIP Tier</label>
            <select v-model="editUser.vip_level" class="w-full p-4 bg-gray-50 border border-gray-100 rounded-xl font-black text-gray-900 outline-none">
              <option :value="0">普通用户</option>
              <option :value="1">银卡会员</option>
              <option :value="2">金卡会员</option>
              <option :value="3">钻石会员</option>
            </select>
          </div>
        </div>

        <div class="flex space-x-4 mt-8">
          <button @click="editUser = null" class="flex-1 py-4 bg-gray-100 text-gray-400 rounded-xl font-black text-xs uppercase tracking-widest hover:bg-gray-200 transition-all">Cancel</button>
          <button @click="saveUserChanges" class="flex-1 py-4 bg-gray-900 text-white rounded-xl font-black text-xs uppercase tracking-widest hover:bg-indigo-600 shadow-lg shadow-indigo-100 transition-all">Save Changes</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';
const users = ref([]);
const editUser = ref(null);

const fetchUsers = async () => {
  try {
    const res = await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/consumers');
    const data = await res.json();
    if (data.success) users.value = data.data;
  } catch (error) {
    console.error('Failed to load users');
  }
};

const formatTime = (time) => {
  if (!time) return 'N/A';
  return new Date(time).toLocaleDateString();
};

const getVipName = (level) => {
  const map = { 0: 'Standard', 1: 'Silver', 2: 'Gold', 3: 'Diamond' };
  return map[level] || 'Unknown';
};

const getVipClass = (level) => {
  if (level === 3) return 'bg-purple-100 text-purple-700 border border-purple-200';
  if (level === 2) return 'bg-yellow-100 text-yellow-700 border border-yellow-200';
  if (level === 1) return 'bg-blue-100 text-blue-700 border border-blue-200';
  return 'bg-gray-100 text-gray-500 border border-gray-200';
};

const openEditBalance = (user) => {
  editUser.value = { ...user };
};

const saveUserChanges = async () => {
  try {
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/admin/consumers/${editUser.value.consumer_id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(editUser.value)
    });
    if ((await res.json()).success) {
      editUser.value = null;
      fetchUsers();
    }
  } catch (error) {
    alert('Failed to update');
  }
};

// 🌟 核心：带业务检查的删除 (3.2.1)
const deleteUser = async (id) => {
  if (!confirm('Are you sure? This action is permanent.')) return;
  try {
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/admin/consumers/${id}`, { method: 'DELETE' });
    const data = await res.json();
    if (data.success) {
      fetchUsers();
    } else {
      // 🌟 如果后端返回“有订单不能删”，这里会弹出提示
      alert(data.message);
    }
  } catch (error) {
    alert('System error during deletion');
  }
};

onMounted(fetchUsers);
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