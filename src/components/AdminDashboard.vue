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

    <!-- 👇 新增：高级数据分析看板 👇 -->
    <div class="analysis-container">
      <!-- 1. 品类销售效能大盘 -->
      <div class="analysis-card">
        <h3>📚 品类销售效能大盘</h3>
        <table class="data-table">
          <thead>
          <tr>
            <th>图书分类</th>
            <th>累计销量 (本)</th>
            <th>创造营收 (元)</th>
            <th>预估毛利 (元)</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(item, index) in categoryAnalysis" :key="index">
            <td>{{ item.category }}</td>
            <td>{{ item.total_qty }}</td>
            <td>¥{{ Number(item.total_amount).toFixed(2) }}</td>
            <td class="highlight-profit">¥{{ (item.total_amount * 0.3).toFixed(2) }}</td>
          </tr>
          <tr v-if="categoryAnalysis.length === 0">
            <td colspan="4" class="empty-text">暂无品类销售数据</td>
          </tr>
          </tbody>
        </table>
      </div>

      <!-- 2. 会员消费力分析 -->
      <div class="analysis-card">
        <h3>👑 会员消费力分析</h3>
        <table class="data-table">
          <thead>
          <tr>
            <th>会员等级</th>
            <th>等级人数</th>
            <th>贡献总营收 (元)</th>
            <th>平均客单价 (元)</th>
          </tr>
          </thead>
          <tbody>
          <tr v-for="(vip, index) in vipAnalysis" :key="index">
            <td>
              <span v-if="vip.vip_level === 3" class="vip-tag diamond">钻石会员</span>
              <span v-else-if="vip.vip_level === 2" class="vip-tag gold">金卡会员</span>
              <span v-else-if="vip.vip_level === 1" class="vip-tag silver">银卡会员</span>
              <span v-else class="vip-tag normal">普通用户</span>
            </td>
            <td>{{ vip.user_count }} 人</td>
            <td>¥{{ Number(vip.total_revenue || 0).toFixed(2) }}</td>
            <td>¥{{ vip.user_count > 0 ? (vip.total_revenue / vip.user_count).toFixed(2) : 0 }}</td>
          </tr>
          <tr v-if="vipAnalysis.length === 0">
            <td colspan="4" class="empty-text">暂无会员消费数据</td>
          </tr>
          </tbody>
        </table>
      </div>
    </div>
    <!-- 👆 新增结束 👆 -->

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

// 🌟新增：定义存放分析数据的响应式变量
const categoryAnalysis = ref([]);
const vipAnalysis = ref([]);

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

// 🌟新增：获取品类销售数据
const fetchCategoryAnalysis = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/analysis/category')).json();
    if (res.success) {
      categoryAnalysis.value = res.data;
    }
  } catch (error) {
    console.error('获取品类分析失败', error);
  }
};

// 🌟新增：获取会员消费力数据
const fetchVipAnalysis = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/analysis/vip')).json();
    if (res.success) {
      vipAnalysis.value = res.data;
    }
  } catch (error) {
    console.error('获取会员分析失败', error);
  }
};

onMounted(() => {
  fetchDashboardStats();
  // 🌟新增：在挂载时同时请求另外两个图表的数据
  fetchCategoryAnalysis();
  fetchVipAnalysis();
});
</script>

<style scoped>
/* 确保背景不被影响 */

/* 🌟新增以下全部分析看板样式 */
.analysis-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-top: 30px;
}

.analysis-card {
  flex: 1;
  min-width: 400px;
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(16px);
  padding: 24px;
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-radius: 2rem;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.05);
}

.analysis-card h3 {
  margin-top: 0;
  color: #111827;
  border-bottom: 2px solid rgba(0, 0, 0, 0.05);
  padding-bottom: 12px;
  font-size: 18px;
  font-weight: 800;
  letter-spacing: -0.025em;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
  font-size: 14px;
  font-weight: 500;
}

.data-table th, .data-table td {
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  padding: 14px 10px;
  text-align: left;
  color: #374151;
}

.data-table th {
  color: #9ca3af;
  font-weight: 800;
  text-transform: uppercase;
  font-size: 12px;
  letter-spacing: 0.05em;
}

.highlight-profit {
  color: #ef4444 !important;
  font-weight: 800;
}

.empty-text {
  text-align: center;
  color: #9ca3af;
  padding: 30px 0;
  font-weight: bold;
}

/* VIP标签美化 */
.vip-tag {
  padding: 6px 10px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
.diamond { background: #e0e7ff; color: #4338ca; border: 1px solid #c7d2fe; }
.gold { background: #fef3c7; color: #b45309; border: 1px solid #fde68a; }
.silver { background: #f3f4f6; color: #4b5563; border: 1px solid #e5e7eb; }
.normal { background: #f9fafb; color: #9ca3af; }
</style>