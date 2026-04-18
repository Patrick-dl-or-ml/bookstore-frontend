<template>
  <div class="max-w-5xl mx-auto py-12 px-6 font-sans relative min-h-screen">

    <div class="absolute top-0 left-0 w-96 h-96 bg-purple-100/50 rounded-full blur-[100px] -z-10"></div>

    <header class="mb-12 flex items-center justify-between">
      <div>
        <h1 class="text-4xl font-black text-gray-900 tracking-tighter mb-2">My Account</h1>
        <p class="text-sm font-bold text-gray-400 uppercase tracking-widest">Manage your profile, orders, and addresses</p>
      </div>
      <div class="flex flex-col items-end">
        <span class="px-4 py-1.5 bg-gradient-to-r from-indigo-500 to-purple-500 text-white text-xs font-black uppercase tracking-widest rounded-xl shadow-lg">
          VIP Level {{ userInfo.vip_level || 0 }}
        </span>
        <span class="text-sm font-bold text-gray-500 mt-2">Integral: <span class="text-indigo-600 font-black">{{ userInfo.integral || 0 }}</span></span>
      </div>
    </header>

    <div class="flex space-x-8 border-b border-gray-200 mb-10 overflow-x-auto custom-scrollbar">
      <button @click="activeTab = 'profile'"
              :class="activeTab === 'profile' ? 'border-indigo-600 text-indigo-600' : 'border-transparent text-gray-400 hover:text-gray-900'"
              class="pb-4 text-sm font-black uppercase tracking-widest border-b-4 transition-all whitespace-nowrap">
        Profile Settings
      </button>
      <button @click="activeTab = 'orders'"
              :class="activeTab === 'orders' ? 'border-indigo-600 text-indigo-600' : 'border-transparent text-gray-400 hover:text-gray-900'"
              class="pb-4 text-sm font-black uppercase tracking-widest border-b-4 transition-all whitespace-nowrap">
        Order History
      </button>
      <button @click="activeTab = 'addresses'"
              :class="activeTab === 'addresses' ? 'border-indigo-600 text-indigo-600' : 'border-transparent text-gray-400 hover:text-gray-900'"
              class="pb-4 text-sm font-black uppercase tracking-widest border-b-4 transition-all whitespace-nowrap">
        Shipping Addresses
      </button>
    </div>

    <Transition name="fade" mode="out-in">

      <div v-if="activeTab === 'profile'" class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-10 shadow-xl">
        <h2 class="text-2xl font-black text-gray-900 mb-8">Personal Information</h2>
        <form @submit.prevent="updateProfile" class="space-y-6 max-w-2xl">
          <div class="space-y-2">
            <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Display Name</label>
            <input v-model="editForm.consumer_name" type="text" class="w-full p-5 bg-white border border-gray-100 rounded-2xl font-bold text-gray-900 focus:ring-4 focus:ring-indigo-100 outline-none transition-all" />
          </div>
          <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div class="space-y-2">
              <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Email Address</label>
              <input v-model="editForm.email" type="email" class="w-full p-5 bg-white border border-gray-100 rounded-2xl font-bold text-gray-900 focus:ring-4 focus:ring-indigo-100 outline-none transition-all" />
            </div>
            <div class="space-y-2">
              <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Phone Number</label>
              <input v-model="editForm.phone" type="tel" class="w-full p-5 bg-white border border-gray-100 rounded-2xl font-bold text-gray-900 focus:ring-4 focus:ring-indigo-100 outline-none transition-all" />
            </div>
          </div>
          <div class="pt-6">
            <button type="submit" :disabled="isSaving" class="px-10 py-5 bg-gray-900 text-white rounded-2xl font-black text-sm uppercase tracking-widest shadow-xl hover:bg-indigo-600 transition-all active:scale-95 disabled:opacity-50">
              {{ isSaving ? 'Saving...' : 'Update Profile' }}
            </button>
          </div>
        </form>
      </div>

      <div v-else-if="activeTab === 'orders'" class="space-y-6">
        <div v-if="myOrders.length === 0" class="py-20 text-center bg-white/40 rounded-[3rem] border border-dashed border-gray-300">
          <p class="text-gray-400 text-lg font-black uppercase tracking-widest">You have no past orders.</p>
        </div>
        <div v-for="order in myOrders" :key="order.sale_id" class="bg-white/80 backdrop-blur-md border border-white rounded-[2rem] p-8 shadow-sm hover:shadow-lg transition-all">
          <div class="flex flex-col md:flex-row md:items-center justify-between border-b border-gray-100 pb-6 mb-6">
            <div>
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">Order Placed</p>
              <p class="text-sm font-bold text-gray-900">{{ new Date(order.order_time).toLocaleString() }}</p>
            </div>
            <div class="mt-4 md:mt-0 text-left md:text-right">
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">Order Total</p>
              <p class="text-2xl font-black text-gray-900 italic">¥{{ order.total_price }}</p>
            </div>
          </div>
          <div class="flex items-center justify-between">
            <div class="flex space-x-3">
               <span :class="order.payment_status === '已支付' ? 'bg-emerald-100 text-emerald-700' : 'bg-amber-100 text-amber-700'" class="px-4 py-2 rounded-xl text-[10px] font-black uppercase tracking-widest">
                {{ order.payment_status || '未支付' }}
              </span>
              <span :class="order.delivery_status === '已发货' ? 'bg-blue-100 text-blue-700' : (order.delivery_status === '已签收' ? 'bg-emerald-100 text-emerald-700' : 'bg-gray-100 text-gray-600')" class="px-4 py-2 rounded-xl text-[10px] font-black uppercase tracking-widest">
                {{ order.delivery_status || '未发货' }}
              </span>
            </div>
          </div>
        </div>
      </div>

      <div v-else-if="activeTab === 'addresses'" class="space-y-6">

        <div class="flex justify-between items-center bg-white/60 backdrop-blur-xl border border-white rounded-[2rem] p-6 shadow-sm">
          <h2 class="text-xl font-black text-gray-900">Saved Addresses</h2>
          <button class="px-6 py-3 bg-gray-900 text-white rounded-xl font-black text-xs uppercase tracking-widest hover:bg-indigo-600 transition-all active:scale-95 shadow-md">
            + Add New
          </button>
        </div>

        <div v-if="myAddresses.length === 0" class="py-20 text-center bg-white/40 rounded-[3rem] border border-dashed border-gray-300">
          <p class="text-gray-400 text-lg font-black uppercase tracking-widest">No shipping addresses saved yet.</p>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div v-for="addr in myAddresses" :key="addr.address_id"
               class="group relative bg-white/80 backdrop-blur-md border rounded-[2rem] p-8 transition-all"
               :class="addr.is_default ? 'border-indigo-400 shadow-md ring-4 ring-indigo-50' : 'border-white hover:border-gray-200 shadow-sm'">

            <div v-if="addr.is_default" class="absolute top-6 right-6">
              <span class="px-3 py-1 bg-indigo-100 text-indigo-700 text-[9px] font-black uppercase tracking-widest rounded-lg">Default</span>
            </div>

            <div class="mb-6 pr-16">
              <h3 class="text-xl font-black text-gray-900 mb-1">{{ addr.receiver_name }}</h3>
              <p class="text-sm font-bold text-gray-500 font-mono">{{ addr.receiver_phone }}</p>
            </div>

            <div class="text-sm font-bold text-gray-700 leading-relaxed mb-8">
              <p>{{ addr.province }} {{ addr.city }} {{ addr.district }}</p>
              <p class="text-gray-500">{{ addr.detail_address }}</p>
            </div>

            <div class="flex items-center space-x-4 border-t border-gray-100 pt-4 opacity-0 group-hover:opacity-100 transition-opacity">
              <button class="text-xs font-black text-indigo-600 uppercase tracking-widest hover:text-indigo-800">Edit</button>
              <span class="text-gray-300">|</span>
              <button class="text-xs font-black text-red-400 uppercase tracking-widest hover:text-red-600">Delete</button>

              <div class="flex-1 text-right">
                <button v-if="!addr.is_default" @click="setDefaultAddress(addr.address_id)" class="text-[10px] font-black text-gray-400 uppercase tracking-widest hover:text-gray-900 transition-colors">
                  Set as Default
                </button>
              </div>
            </div>
          </div>
        </div>

      </div>

    </Transition>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const activeTab = ref('profile');
const userInfo = ref({});
const editForm = ref({ consumer_name: '', email: '', phone: '' });
const myOrders = ref([]);
const myAddresses = ref([]); // 🌟 新增：存放地址的数组
const isSaving = ref(false);

const getCurrentUserId = () => {
  const user = JSON.parse(localStorage.getItem('user') || '{}');
  return user.id || user.consumer_id;
};

const fetchUserProfile = async () => {
  const userId = getCurrentUserId();
  if (!userId) return;
  try {
    const res = await (await fetch(`https://bookstore-backend-60vr.onrender.com/${userId}/profile`)).json();
    if (res.success && res.data) {
      userInfo.value = res.data;
      editForm.value = { consumer_name: res.data.consumer_name, email: res.data.email, phone: res.data.phone };
      const localUser = JSON.parse(localStorage.getItem('user') || '{}');
      localUser.name = res.data.consumer_name;
      localStorage.setItem('user', JSON.stringify(localUser));
    }
  } catch (err) { console.error('Failed to fetch profile'); }
};

const updateProfile = async () => {
  const userId = getCurrentUserId();
  if (!userId) return;
  isSaving.value = true;
  try {
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/${userId}/profile`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(editForm.value)
    });
    if ((await res.json()).success) {
      alert('Profile updated successfully!');
      fetchUserProfile();
    }
  } catch (err) { alert('Update failed'); }
  finally { isSaving.value = false; }
};

const fetchMyOrders = async () => {
  const userId = getCurrentUserId();
  if (!userId) return;
  try {
    const res = await (await fetch(`https://bookstore-backend-60vr.onrender.com/${userId}/orders`)).json();
    if (res.success) myOrders.value = res.data;
  } catch (err) { console.error('Failed to fetch orders'); }
};

// 🌟 核心新功能：获取我的地址簿
const fetchMyAddresses = async () => {
  const userId = getCurrentUserId();
  if (!userId) return;
  try {
    const res = await (await fetch(`https://bookstore-backend-60vr.onrender.com/${userId}/addresses`)).json();
    if (res.success) myAddresses.value = res.data;
  } catch (err) { console.error('Failed to fetch addresses'); }
};

// 🌟 设为默认地址的逻辑 (前端展示用，你需要配对应的后端接口)
const setDefaultAddress = async (addressId) => {
  const userId = getCurrentUserId();
  try {
    // 假设你有一个设置默认地址的接口
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/api/users/${userId}/addresses/${addressId}/default`, {
      method: 'PUT'
    });
    if ((await res.json()).success) {
      fetchMyAddresses(); // 刷新地址列表，让 UI 上的 Default 徽章移动
    }
  } catch (err) { alert('Operation failed'); }
};

onMounted(() => {
  fetchUserProfile();
  fetchMyOrders();
  fetchMyAddresses(); // 页面加载时顺便拉取地址
});
</script>

<style scoped>
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s ease, transform 0.3s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(10px); }
.custom-scrollbar::-webkit-scrollbar { display: none; }
</style>