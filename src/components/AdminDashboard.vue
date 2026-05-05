<template>
  <div class="max-w-7xl mx-auto py-10 px-6 font-sans relative min-h-screen">

    <!-- 你写好的炫酷背景光晕完全保留 -->
    <div class="absolute top-10 right-20 w-[500px] h-[500px] bg-indigo-200/30 rounded-full blur-[120px] -z-10 animate-pulse"></div>
    <div class="absolute bottom-20 left-10 w-[400px] h-[400px] bg-emerald-200/20 rounded-full blur-[100px] -z-10"></div>

    <header class="mb-12">
      <h1 class="text-4xl font-black text-gray-900 tracking-tighter mb-2">Executive Dashboard</h1>
      <p class="text-sm font-bold text-gray-400 uppercase tracking-widest">Real-time business performance & insights</p>
    </header>

    <!-- 🌟 修复关键：把 3.2.3 和 3.2.4 的所有内容都放进了 main 里，确保有数据才渲染 -->
    <main v-if="!isLoading" class="space-y-12 pb-20">

      <!-- 顶部基础统计卡片 (原封不动保留) -->
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

      <!-- 👇 3.2.3 经营数据实时统计大盘 (完全保留你的设计) 👇 -->
      <section class="mt-16">
        <div class="flex items-center space-x-3 mb-8">
          <div class="h-8 w-2 bg-indigo-600 rounded-full"></div>
          <h2 class="text-2xl font-black text-gray-900 tracking-tight">3.2.3 经营数据实时统计</h2>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
          <!-- 畅销图书榜 (3.2.3.3) -->
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
              <!-- 缺省态占位 -->
              <div v-if="topBooks.length === 0" class="py-6 text-center text-gray-400 text-xs font-bold">暂无畅销书数据</div>
            </div>
          </div>

          <!-- 核心客户榜 (3.2.3.2) -->
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
              <div v-if="topCustomers.length === 0" class="py-6 text-center text-gray-400 text-xs font-bold">暂无客户消费数据</div>
            </div>
          </div>

          <!-- 物流履约分布 (3.2.3.5) -->
          <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm flex flex-col justify-between">
            <h3 class="text-sm font-black text-gray-400 uppercase tracking-widest mb-6">🚚 Logistics Status</h3>
            <div class="grid grid-cols-2 gap-4">
              <div v-for="item in logisticsStats" :key="item.delivery_status" class="p-4 border border-gray-100 rounded-2xl text-center">
                <p class="text-2xl font-black text-gray-900">{{ item.count }}</p>
                <p class="text-[9px] font-bold text-gray-400 uppercase tracking-widest mt-1">{{ item.delivery_status }}</p>
              </div>
            </div>
            <div v-if="logisticsStats.length === 0" class="py-6 text-center text-gray-400 text-xs font-bold">暂无物流数据</div>
          </div>
        </div>
      </section>

      <!-- 👇 3.2.4 销售深度关联分析看板 (完全保留你的高颜值表格设计) 👇 -->
      <section class="mt-16">
        <div class="flex items-center space-x-3 mb-8">
          <div class="h-8 w-2 bg-purple-600 rounded-full"></div>
          <h2 class="text-2xl font-black text-gray-900 tracking-tight">3.2.4 销售深度关联分析</h2>
        </div>

        <div class="grid grid-cols-1 xl:grid-cols-2 gap-8">
          <!-- 1. 品类销售效能大盘 -->
          <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm">
            <h3 class="text-sm font-black text-gray-400 uppercase tracking-widest mb-6">📚 品类销售效能</h3>
            <table class="w-full text-left">
              <thead>
              <tr class="text-[10px] font-black text-gray-400 uppercase tracking-widest border-b border-gray-100">
                <th class="pb-4">图书分类</th>
                <th class="pb-4 text-center">累计销量</th>
                <th class="pb-4 text-right">创造营收</th>
              </tr>
              </thead>
              <tbody class="divide-y divide-gray-50">
              <tr v-for="(item, index) in categoryAnalysis" :key="index" class="group">
                <td class="py-4 text-sm font-bold text-gray-900">{{ item.category }}</td>
                <td class="py-4 text-sm font-black text-gray-600 text-center">{{ item.total_qty }}</td>
                <td class="py-4 text-sm font-black text-indigo-600 text-right">¥{{ formatMoney(item.total_amount) }}</td>
              </tr>
              <tr v-if="categoryAnalysis.length === 0">
                <td colspan="3" class="py-10 text-center text-gray-400 font-bold text-xs">暂无品类销售数据</td>
              </tr>
              </tbody>
            </table>
          </div>

          <!-- 2. 会员消费力分析 -->
          <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm">
            <h3 class="text-sm font-black text-gray-400 uppercase tracking-widest mb-6">👑 会员消费力与资产分析</h3>
            <table class="w-full text-left">
              <thead>
              <tr class="text-[10px] font-black text-gray-400 uppercase tracking-widest border-b border-gray-100">
                <th class="pb-4">等级</th>
                <th class="pb-4 text-right">营收贡献</th>
                <th class="pb-4 text-right">平均余额</th>
              </tr>
              </thead>
              <tbody class="divide-y divide-gray-50">
              <tr v-for="(vip, index) in vipAnalysis" :key="index">
                <td class="py-4">
                  <!-- 你的徽章样式完全保留！ -->
                  <span :class="getVipTagClass(vip.vip_level)" class="px-2 py-1 rounded text-[9px] font-black uppercase tracking-tighter">
                      {{ getVipName(vip.vip_level) }}
                    </span>
                </td>
                <td class="py-4 text-sm font-black text-gray-900 text-right">¥{{ formatMoney(vip.total_revenue) }}</td>
                <td class="py-4 text-sm font-black text-blue-600 text-right">¥{{ formatMoney(vip.avg_balance) }}</td>
              </tr>
              <tr v-if="vipAnalysis.length === 0">
                <td colspan="3" class="py-10 text-center text-gray-400 font-bold text-xs">暂无会员消费数据</td>
              </tr>
              </tbody>
            </table>
          </div>

          <!-- 3. 大额订单雷达 -->
          <div class="xl:col-span-2 bg-gray-900 rounded-[2.5rem] p-8 shadow-2xl text-white">
            <div class="flex items-center justify-between mb-8">
              <div>
                <h3 class="text-lg font-black tracking-tight">大额重点订单监控 (Radar)</h3>
                <p class="text-[10px] font-bold text-gray-500 uppercase tracking-[0.2em]">Monitoring orders > ¥500.00</p>
              </div>
              <div class="px-4 py-2 bg-white/10 rounded-xl text-[10px] font-black">LIVE UPDATE</div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
              <div v-for="order in bigOrders" :key="order.sale_id" class="p-5 bg-white/5 border border-white/10 rounded-2xl hover:bg-white/10 transition-all">
                <p class="text-[9px] font-black text-indigo-400 mb-1">ID #{{ String(order.sale_id).padStart(5, '0') }}</p>
                <p class="text-xl font-black mb-1">¥{{ formatMoney(order.total_price) }}</p>
                <p class="text-[10px] font-bold text-gray-500 italic">{{ order.consumer_name }}</p>
              </div>
              <div v-if="bigOrders.length === 0" class="col-span-4 py-6 text-center text-gray-600 text-xs font-bold">
                雷达扫描中：暂未发现异常大额订单
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

// 格式化金额
const formatMoney = (val) => {
  return Number(val || 0).toLocaleString('zh-CN', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
};

// 🌟 修复关键：补齐缺失的 VIP 转换函数，否则页面会崩溃！
const getVipName = (level) => {
  const levels = { 3: '钻石会员', 2: '金卡会员', 1: '银卡会员', 0: '普通用户' };
  return levels[level] || '普通用户';
};

// 🌟 修复关键：补齐缺失的 VIP 样式函数
const getVipTagClass = (level) => {
  if (level === 3) return 'bg-purple-100 text-purple-700 border border-purple-200';
  if (level === 2) return 'bg-yellow-100 text-yellow-700 border border-yellow-200';
  if (level === 1) return 'bg-blue-100 text-blue-700 border border-blue-200';
  return 'bg-gray-100 text-gray-500 border border-gray-200';
};

// 各种拉取数据的 API 请求
const fetchDashboardStats = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/dashboard/stats')).json();
    if (res.success) stats.value = res.data;
  } catch (error) { console.error('Failed to load dashboard data'); }
};

const fetchRankings = async () => {
  try {
    const [booksRes, customersRes, logisticsRes] = await Promise.all([
      fetch('https://bookstore-backend-60vr.onrender.com/api/admin/analysis/top-books').then(r => r.json()),
      fetch('https://bookstore-backend-60vr.onrender.com/api/admin/analysis/top-customers').then(r => r.json()),
      fetch('https://bookstore-backend-60vr.onrender.com/api/admin/analysis/logistics').then(r => r.json())
    ]);

    if (booksRes.success) topBooks.value = booksRes.data;
    if (customersRes.success) topCustomers.value = customersRes.data;
    if (logisticsRes.success) logisticsStats.value = logisticsRes.data;
  } catch (error) { console.error('获取排行统计失败', error); }
};

const fetchDeepAnalysis = async () => {
  try {
    const [catRes, vipRes] = await Promise.all([
      fetch('https://bookstore-backend-60vr.onrender.com/api/admin/analysis/category').then(r => r.json()),
      fetch('https://bookstore-backend-60vr.onrender.com/api/admin/analysis/vip').then(r => r.json())
    ]);

    if (catRes.success) categoryAnalysis.value = catRes.data;
    if (vipRes.success) vipAnalysis.value = vipRes.data;
  } catch (error) { console.error('获取深度分析失败', error); }
};

const fetchBigOrders = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/orders')).json();
    if (res.success) {
      bigOrders.value = res.data.filter(order => order.total_price > 500).slice(0, 5);
    }
  } catch (error) { console.error('获取大额订单失败', error); }
};

onMounted(async () => {
  isLoading.value = true;
  await Promise.all([
    fetchDashboardStats(),
    fetchRankings(),
    fetchDeepAnalysis(),
    fetchBigOrders()
  ]);
  isLoading.value = false;
});
</script>

<style scoped>
/* 🌟 修复关键：清理了会导致 CSS 编译瘫痪的换行，保留你最精简完美的 Tailwind 逻辑 */
.analysis-container {
  @apply mt-12 grid grid-cols-1 xl:grid-cols-2 gap-8;
}

.analysis-card {
  @apply bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm transition-all hover:shadow-md;
}

.analysis-card h3 {
  @apply text-sm font-black text-gray-400 uppercase tracking-widest mb-6 flex items-center;
}

.data-table {
  @apply w-full text-left border-collapse;
}

.data-table th {
  @apply pb-4 text-[10px] font-black text-gray-400 uppercase tracking-widest border-b border-gray-100;
}

.data-table td {
  @apply py-4 text-sm font-bold text-gray-900 border-b border-gray-50/50;
}
</style>