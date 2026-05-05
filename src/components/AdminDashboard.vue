<template>
  <div class="max-w-7xl mx-auto py-10 px-6 font-sans relative min-h-screen">

    <!-- Ambient Background Accents -->
    <div class="absolute top-10 right-20 w-[500px] h-[500px] bg-indigo-200/30 rounded-full blur-[120px] -z-10 animate-pulse"></div>
    <div class="absolute bottom-20 left-10 w-[400px] h-[400px] bg-emerald-200/20 rounded-full blur-[100px] -z-10"></div>

    <header class="mb-12">
      <h1 class="text-4xl font-black text-gray-900 tracking-tighter mb-2">Executive Dashboard</h1>
      <p class="text-sm font-bold text-gray-400 uppercase tracking-widest">Real-time business performance & insights</p>
    </header>

    <main v-if="!isLoading" class="space-y-12 pb-20">

      <!-- Top Metric Cards -->
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

      <!-- 3.2.3 Real-time Operations -->
      <section class="mt-16">
        <div class="flex items-center space-x-3 mb-8">
          <div class="h-8 w-2 bg-indigo-600 rounded-full"></div>
          <h2 class="text-2xl font-black text-gray-900 tracking-tight">3.2.3 Real-time Operations</h2>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
          <!-- Top Selling Books -->
          <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm">
            <h3 class="text-sm font-black text-gray-400 uppercase tracking-widest mb-6">🏆 Top Selling Books</h3>
            <div class="space-y-4">
              <div v-for="(book, index) in topBooks" :key="index" class="flex items-center justify-between p-4 bg-gray-50/50 rounded-2xl">
                <div class="flex items-center space-x-3">
                  <span class="text-xs font-black text-indigo-400">0{{ index + 1 }}</span>
                  <p class="text-sm font-bold text-gray-900 truncate w-32">{{ book.book_name }}</p>
                </div>
                <p class="text-xs font-black text-gray-400">{{ book.total_sold }} Units</p>
              </div>
              <div v-if="topBooks.length === 0" class="py-6 text-center text-gray-400 text-xs font-bold">No records found</div>
            </div>
          </div>

          <!-- Core Customers -->
          <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm">
            <h3 class="text-sm font-black text-gray-400 uppercase tracking-widest mb-6">👑 Core Customers</h3>
            <div class="space-y-4">
              <div v-for="(user, index) in topCustomers" :key="index" class="flex items-center justify-between p-4 bg-gray-50/50 rounded-2xl">
                <div class="flex items-center space-x-3">
                  <div class="w-2 h-2 rounded-full bg-emerald-400"></div>
                  <p class="text-sm font-bold text-gray-900">{{ user.consumer_name }}</p>
                </div>
                <p class="text-xs font-black text-emerald-600">¥{{ formatMoney(user.total_spent) }}</p>
              </div>
              <div v-if="topCustomers.length === 0" class="py-6 text-center text-gray-400 text-xs font-bold">No records found</div>
            </div>
          </div>

          <!-- Logistics Status -->
          <!-- Logistics Status -->
          <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm">
            <h3 class="text-sm font-black text-gray-400 uppercase tracking-widest mb-6">🚚 Logistics Status</h3>
            <div class="space-y-4">
              <div v-for="item in logisticsStats" :key="item.delivery_status" class="flex items-center justify-between p-4 bg-gray-50/50 rounded-2xl hover:bg-white transition-all shadow-sm">
                <div class="flex items-center space-x-3">
                  <!-- 动态状态指示灯 -->
                  <div class="w-2 h-2 rounded-full" :class="getStatusColor(item.delivery_status)"></div>
                  <p class="text-xs font-bold text-gray-900 uppercase tracking-widest">{{ translateStatus(item.delivery_status) }}</p>
                </div>
                <div class="flex items-baseline space-x-1">
                  <p class="text-xl font-black text-gray-900">{{ item.count }}</p>
                  <p class="text-[9px] font-bold text-gray-400 uppercase">Orders</p>
                </div>
              </div>
              <div v-if="logisticsStats.length === 0" class="py-6 text-center text-gray-400 text-xs font-bold">No records found</div>
            </div>
          </div>
        </div>
      </section>

      <!-- 3.2.4 Advanced Sales Analytics -->
      <section class="mt-16">
        <div class="flex items-center space-x-3 mb-8">
          <div class="h-8 w-2 bg-purple-600 rounded-full"></div>
          <h2 class="text-2xl font-black text-gray-900 tracking-tight">3.2.4 Advanced Sales Analytics</h2>
        </div>

        <div class="grid grid-cols-1 xl:grid-cols-2 gap-8">
          <!-- Category Performance -->
          <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm">
            <h3 class="text-sm font-black text-gray-400 uppercase tracking-widest mb-6">📚 Category Performance</h3>
            <table class="w-full text-left">
              <thead>
              <tr class="text-[10px] font-black text-gray-400 uppercase tracking-widest border-b border-gray-100">
                <th class="pb-4">Category</th>
                <th class="pb-4 text-center">Qty Sold</th>
                <th class="pb-4 text-right">Revenue</th>
              </tr>
              </thead>
              <tbody class="divide-y divide-gray-50">
              <tr v-for="(item, index) in categoryAnalysis" :key="index" class="group">
                <td class="py-4 text-sm font-bold text-gray-900">{{ item.category_name || item.category || 'Unknown' }}</td>
                <td class="py-4 text-sm font-black text-gray-600 text-center">{{ item.total_qty }}</td>
                <td class="py-4 text-sm font-black text-indigo-600 text-right">¥{{ formatMoney(item.total_amount) }}</td>
              </tr>
              <tr v-if="categoryAnalysis.length === 0">
                <td colspan="3" class="py-10 text-center text-gray-400 font-bold text-xs">No data available</td>
              </tr>
              </tbody>
            </table>
          </div>

          <!-- Customer Value & Assets -->
          <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm">
            <h3 class="text-sm font-black text-gray-400 uppercase tracking-widest mb-6">👑 Customer Value & Assets</h3>
            <table class="w-full text-left">
              <thead>
              <tr class="text-[10px] font-black text-gray-400 uppercase tracking-widest border-b border-gray-100">
                <th class="pb-4">Tier</th>
                <th class="pb-4 text-right">Revenue</th>
                <th class="pb-4 text-right">Avg Balance</th>
              </tr>
              </thead>
              <tbody class="divide-y divide-gray-50">
              <tr v-for="(vip, index) in vipAnalysis" :key="index">
                <td class="py-4">
                  <span :class="getVipTagClass(vip.vip_level)" class="px-2 py-1 rounded text-[9px] font-black uppercase tracking-tighter">
                      {{ getVipName(vip.vip_level) }}
                    </span>
                </td>
                <td class="py-4 text-sm font-black text-gray-900 text-right">¥{{ formatMoney(vip.total_revenue) }}</td>
                <td class="py-4 text-sm font-black text-blue-600 text-right">¥{{ formatMoney(vip.avg_balance) }}</td>
              </tr>
              <tr v-if="vipAnalysis.length === 0">
                <td colspan="3" class="py-10 text-center text-gray-400 font-bold text-xs">No data available</td>
              </tr>
              </tbody>
            </table>
          </div>

          <!-- High-Value Order Radar (Now matching light theme) -->
          <div class="xl:col-span-2 bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm">
            <div class="flex items-center justify-between mb-8">
              <div>
                <h3 class="text-lg font-black tracking-tight text-gray-900">High-Value Order Monitor (Radar)</h3>
                <p class="text-[10px] font-bold text-gray-400 uppercase tracking-[0.2em]">Monitoring orders > ¥500.00</p>
              </div>
              <div class="px-4 py-2 bg-indigo-50 text-indigo-600 rounded-xl text-[10px] font-black animate-pulse">LIVE UPDATE</div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
              <div v-for="order in bigOrders" :key="order.sale_id" class="p-5 bg-white border border-gray-100 rounded-2xl hover:shadow-md transition-all">
                <p class="text-[9px] font-black text-indigo-500 mb-1">ID #{{ String(order.sale_id).padStart(5, '0') }}</p>
                <p class="text-xl font-black text-gray-900 mb-1">¥{{ formatMoney(order.total_price) }}</p>
                <p class="text-[10px] font-bold text-gray-400 italic">{{ order.consumer_name }}</p>
              </div>
              <div v-if="bigOrders.length === 0" class="col-span-4 py-6 text-center text-gray-400 text-xs font-bold">
                Scanning... No high-value orders detected.
              </div>
            </div>
          </div>
        </div>
      </section>

    </main>

    <div v-else class="flex items-center justify-center py-32">
      <div class="w-12 h-12 border-4 border-indigo-200 border-t-indigo-600 rounded-full animate-spin"></div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const isLoading = ref(true);

// 基础统计数据
const stats = ref({
  todayBooks: 0, todayRevenue: 0, monthBooks: 0, monthRevenue: 0, totalUsers: 0, totalBooks: 0
});

// 排行榜和分析数据
const topBooks = ref([]);
const topCustomers = ref([]);
const logisticsStats = ref([]);
const categoryAnalysis = ref([]);
const vipAnalysis = ref([]);
const bigOrders = ref([]);

// 🌟 生产环境适配：切换到你的 Render 后端地址
const API_BASE = 'https://bookstore-backend-60vr.onrender.com';

// 格式化金额
const formatMoney = (val) => {
  return Number(val || 0).toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
};

// 各种拉取数据的 API 请求
const fetchDashboardStats = async () => {
  try {
    const res = await (await fetch(`${API_BASE}/api/admin/dashboard/stats`)).json();
    if (res.success) stats.value = res.data;
  } catch (error) { console.error('Failed to load dashboard data'); }
};

const fetchRankings = async () => {
  try {
    const [booksRes, customersRes, logisticsRes] = await Promise.all([
      fetch(`${API_BASE}/api/admin/analysis/top-books`).then(r => r.json()),
      fetch(`${API_BASE}/api/admin/analysis/top-customers`).then(r => r.json()),
      fetch(`${API_BASE}/api/admin/analysis/logistics`).then(r => r.json())
    ]);

    if (booksRes.success) topBooks.value = booksRes.data;
    if (customersRes.success) topCustomers.value = customersRes.data;
    if (logisticsRes.success) logisticsStats.value = logisticsRes.data;
  } catch (error) { console.error('Failed to load rankings'); }
};

const fetchDeepAnalysis = async () => {
  try {
    const [catRes, vipRes] = await Promise.all([
      fetch(`${API_BASE}/api/admin/analysis/category`).then(r => r.json()),
      fetch(`${API_BASE}/api/admin/analysis/vip`).then(r => r.json())
    ]);

    if (catRes.success) categoryAnalysis.value = catRes.data;
    if (vipRes.success) vipAnalysis.value = vipRes.data;
  } catch (error) { console.error('Failed to load deep analysis'); }
};

const fetchBigOrders = async () => {
  try {
    const res = await (await fetch(`${API_BASE}/api/admin/orders`)).json();
    if (res.success) {
      // 🌟 保持兼容性：同时支持 totalprice 和 total_price
      bigOrders.value = res.data.filter(order => (order.totalprice || order.total_price) > 500).slice(0, 5);
    }
  } catch (error) { console.error('Failed to load large orders'); }
};

// 物流状态中英翻译
const translateStatus = (status) => {
  const map = { '已签收': 'Delivered', '已发货': 'Shipped', '未发货': 'Pending' };
  return map[status] || status;
};

// 物流状态对应的指示灯颜色
const getStatusColor = (status) => {
  const map = { '已签收': 'bg-emerald-400', '已发货': 'bg-blue-400', '未发货': 'bg-yellow-400' };
  return map[status] || 'bg-gray-400';
};

// 会员等级标签的样式函数
const getVipTagClass = (level) => {
  const map = {
    0: 'bg-slate-100 text-slate-600',
    1: 'bg-blue-100 text-blue-600',
    2: 'bg-purple-100 text-purple-600',
    3: 'bg-amber-100 text-amber-600'
  };
  return map[level] || 'bg-slate-100 text-slate-600';
};

// 会员名称映射
const getVipName = (level) => {
  const labels = { 0: '普通', 1: '青铜', 2: '白银', 3: '黄金' };
  return labels[level] || '普通';
};

onMounted(async () => {
  isLoading.value = true;
  try {
    await Promise.allSettled([
      fetchDashboardStats(),
      fetchRankings(),
      fetchDeepAnalysis(),
      fetchBigOrders()
    ]);
  } catch (error) {
    console.error('加载失败:', error);
  } finally {
    // 🌟 无论如何，最后都强制把加载动画关掉
    isLoading.value = false;
  }
});

</script>

<style scoped>
.analysis-container {
  margin-top: 3rem;
  display: grid;
  grid-template-columns: repeat(1, minmax(0, 1fr));
  gap: 2rem;
}

@media (min-width: 1280px) {
  .analysis-container {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

.analysis-card {
  background-color: rgba(255, 255, 255, 0.6);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border: 1px solid white;
  border-radius: 2.5rem;
  padding: 2rem;
  box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.analysis-card:hover {
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}

.analysis-card h3 {
  font-size: 0.875rem;
  font-weight: 900;
  color: #9ca3af;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  margin-bottom: 1.5rem;
  display: flex;
  align-items: center;
}

.data-table {
  width: 100%;
  text-align: left;
  border-collapse: collapse;
}

.data-table th {
  padding-bottom: 1rem;
  font-size: 10px;
  font-weight: 900;
  color: #9ca3af;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  border-bottom: 1px solid #f3f4f6;
}

.data-table td {
  padding-top: 1rem;
  padding-bottom: 1rem;
  font-size: 0.875rem;
  font-weight: 700;
  color: #111827;
  border-bottom: 1px solid rgba(249, 250, 251, 0.5);
}
</style>