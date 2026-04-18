<template>
  <div class="max-w-7xl mx-auto py-10 px-6 font-sans relative min-h-screen">

    <div class="absolute top-10 right-20 w-[500px] h-[500px] bg-indigo-200/30 rounded-full blur-[120px] -z-10 animate-pulse"></div>
    <div class="absolute bottom-20 left-10 w-[400px] h-[400px] bg-emerald-200/20 rounded-full blur-[100px] -z-10"></div>

    <header class="mb-12">
      <h1 class="text-4xl font-black text-gray-900 tracking-tighter mb-2">Executive Dashboard</h1>
      <p class="text-sm font-bold text-gray-400 uppercase tracking-widest">Real-time business performance & insights</p>
    </header>

    <main v-if="!isLoading" class="space-y-12">

      <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
        <div class="relative overflow-hidden bg-gradient-to-br from-indigo-600 to-purple-700 rounded-[2.5rem] p-10 shadow-2xl shadow-indigo-200 text-white group">
          <div class="absolute -right-10 -top-10 w-48 h-48 bg-white/10 rounded-full blur-2xl group-hover:scale-150 transition-transform duration-700"></div>
          <p class="text-xs font-black uppercase tracking-[0.3em] text-indigo-100 mb-4 opacity-80">Today's Sales Volume</p>
          <div class="flex items-end space-x-4">
            <span class="text-7xl font-black tracking-tighter">{{ stats.todayBooks }}</span>
            <span class="text-lg font-bold text-indigo-200 pb-2 uppercase tracking-widest">Books</span>
          </div>
        </div>

        <div class="relative overflow-hidden bg-gradient-to-br from-emerald-500 to-teal-600 rounded-[2.5rem] p-10 shadow-2xl shadow-emerald-200 text-white group">
          <div class="absolute -left-10 -bottom-10 w-48 h-48 bg-white/10 rounded-full blur-2xl group-hover:scale-150 transition-transform duration-700"></div>
          <p class="text-xs font-black uppercase tracking-[0.3em] text-emerald-100 mb-4 opacity-80">Today's Revenue</p>
          <div class="flex items-end space-x-2">
            <span class="text-4xl font-black text-emerald-200 pb-1">¥</span>
            <span class="text-7xl font-black tracking-tighter">{{ formatMoney(stats.todayRevenue) }}</span>
          </div>
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-8">

        <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2rem] p-8 shadow-lg hover:-translate-y-1 transition-all">
          <p class="text-[10px] font-black uppercase tracking-widest text-gray-400 mb-2">This Month's Books</p>
          <p class="text-4xl font-black text-gray-900">{{ stats.monthBooks }} <span class="text-sm font-bold text-gray-400 ml-1">Units</span></p>
        </div>

        <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2rem] p-8 shadow-lg hover:-translate-y-1 transition-all">
          <p class="text-[10px] font-black uppercase tracking-widest text-gray-400 mb-2">This Month's Revenue</p>
          <p class="text-4xl font-black text-gray-900 italic">¥{{ formatMoney(stats.monthRevenue) }}</p>
        </div>

        <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2rem] p-8 shadow-lg hover:-translate-y-1 transition-all flex flex-col justify-between">
          <div>
            <p class="text-[10px] font-black uppercase tracking-widest text-gray-400 mb-2">System Capacity</p>
            <div class="flex justify-between items-end">
              <div>
                <p class="text-2xl font-black text-gray-900">{{ stats.totalUsers }}</p>
                <p class="text-[9px] font-bold uppercase text-gray-400">Clients</p>
              </div>
              <div class="text-right">
                <p class="text-2xl font-black text-gray-900">{{ stats.totalBooks }}</p>
                <p class="text-[9px] font-bold uppercase text-gray-400">Vault Assets</p>
              </div>
            </div>
          </div>
        </div>

      </div>

      <div class="text-center pt-10 border-t border-gray-200/50">
        <p class="text-xs font-bold text-gray-400 uppercase tracking-[0.4em]">Data reflects real-time transactions</p>
      </div>

    </main>

    <div v-else class="flex items-center justify-center py-32">
      <div class="w-12 h-12 border-4 border-indigo-200 border-t-indigo-600 rounded-full animate-spin"></div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const isLoading = ref(true);
const stats = ref({
  todayBooks: 0,
  todayRevenue: 0,
  monthBooks: 0,
  monthRevenue: 0,
  totalUsers: 0,
  totalBooks: 0
});

// 金额格式化小工具，比如把 1234.5 变成 1,234.50
const formatMoney = (val) => {
  return Number(val || 0).toLocaleString('zh-CN', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
};

const fetchDashboardStats = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/dashboard/stats')).json();
    if (res.success) {
      stats.value = res.data;
    }
  } catch (error) {
    console.error('Failed to load dashboard data');
  } finally {
    isLoading.value = false;
  }
};

onMounted(() => {
  fetchDashboardStats();
});
</script>

<style scoped>
/* 确保背景不被影响 */
</style>