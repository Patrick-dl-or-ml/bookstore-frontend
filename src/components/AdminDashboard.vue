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
          </tr>
          </thead>
          <tbody>
          <tr v-for="(item, index) in categoryAnalysis" :key="index">
            <td>{{ item.category }}</td>
            <td>{{ item.total_qty }}</td>
            <td>¥{{ Number(item.total_amount).toFixed(2) }}</td>
          </tr>
          <tr v-if="categoryAnalysis.length === 0">
            <!-- 🌟 这里合并单元格从 4 改成了 3 -->
            <td colspan="3" class="empty-text">暂无品类销售数据</td>
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


    <!-- 👇 3.2.3 经营数据实时统计大盘 👇 -->
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
              <p class="text-xs font-black text-emerald-600">¥{{ user.total_spent }}</p>
            </div>
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
        </div>

      </div>
    </section>

    <!-- 👇 3.2.4 销售深度关联分析看板 👇 -->
    <section class="mt-16">
      <div class="flex items-center space-x-3 mb-8">
        <div class="h-8 w-2 bg-purple-600 rounded-full"></div>
        <h2 class="text-2xl font-black text-gray-900 tracking-tight">3.2.4 销售深度关联分析</h2>
      </div>

      <div class="grid grid-cols-1 xl:grid-cols-2 gap-8">
        <!-- 1. 品类销售效能大盘 (保持不变) -->
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
            </tbody>
          </table>
        </div>

        <!-- 2. 会员消费力分析 (补全平均余额) -->
        <div class="bg-white/60 backdrop-blur-xl border border-white rounded-[2.5rem] p-8 shadow-sm">
          <h3 class="text-sm font-black text-gray-400 uppercase tracking-widest mb-6">👑 会员消费力与资产分析</h3>
          <table class="w-full text-left">
            <thead>
            <tr class="text-[10px] font-black text-gray-400 uppercase tracking-widest border-b border-gray-100">
              <th class="pb-4">等级</th>
              <th class="pb-4 text-right">营收贡献</th>
              <th class="pb-4 text-right">平均余额</th> <!-- 🌟 新增列 -->
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
              <td class="py-4 text-sm font-black text-blue-600 text-right">¥{{ formatMoney(vip.avg_balance) }}</td> <!-- 🌟 对应 3.2.1 -->
            </tr>
            </tbody>
          </table>
        </div>

        <!-- 🌟 3. 新增：3.2.4.1 大额订单雷达 (Big Order Radar) -->
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

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const isLoading = ref(true);

// 基础统计数据 (3.2.3 概览)
const stats = ref({
  todayBooks: 0,
  todayRevenue: 0,
  monthBooks: 0,
  monthRevenue: 0,
  totalUsers: 0,
  totalBooks: 0
});

// --- 3.2.3 经营数据统计相关 ---
const topBooks = ref([]);       // 畅销图书排行
const topCustomers = ref([]);   // 核心客户排行
const logisticsStats = ref([]); // 物流履约分布

// --- 3.2.4 深度关联分析相关 ---
const categoryAnalysis = ref([]); // 品类销售效能
const vipAnalysis = ref([]);      // 会员消费力分析
const bigOrders = ref([]);        // 🌟 新增：大额订单雷达 (金额 > 500)

const formatMoney = (val) => {
  return Number(val || 0).toLocaleString('zh-CN', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
};

// 基础看板数据请求
const fetchDashboardStats = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/dashboard/stats')).json();
    if (res.success) stats.value = res.data;
  } catch (error) {
    console.error('Failed to load dashboard data');
  } finally {
    isLoading.value = false;
  }
};

// 3.2.3: 获取图书与客户排行
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
  } catch (error) {
    console.error('获取排行统计失败', error);
  }
};

// 3.2.4: 获取品类与会员深度分析
const fetchDeepAnalysis = async () => {
  try {
    const [catRes, vipRes] = await Promise.all([
      fetch('https://bookstore-backend-60vr.onrender.com/api/admin/analysis/category').then(r => r.json()),
      fetch('https://bookstore-backend-60vr.onrender.com/api/admin/analysis/vip').then(r => r.json())
    ]);

    if (catRes.success) categoryAnalysis.value = catRes.data;
    if (vipRes.success) vipAnalysis.value = vipRes.data;
  } catch (error) {
    console.error('获取深度分析失败', error);
  }
};

// 🌟 3.2.4: 大额订单监控逻辑 (过滤金额 > 500 的单子)
const fetchBigOrders = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/orders')).json();
    if (res.success) {
      // 筛选出符合文档 3.2.4 定义的“大额订单”
      bigOrders.value = res.data.filter(order => order.total_price > 500).slice(0, 5);
    }
  } catch (error) {
    console.error('获取大额订单失败', error);
  }
};

onMounted(() => {
  fetchDashboardStats(); // 基础概览
  fetchRankings();       // 3.2.3 统计
  fetchDeepAnalysis();   // 3.2.4 分析
  fetchBigOrders();      // 3.2.4 监控
});
</script>

<style scoped>
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

.vip-tag {
  padding: 6px 10px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

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
  @apply py-4 text-sm font
  -bold text-gray-900 border-b border-gray-50/50;
}

.vip-tag {
  @apply px-2 py-1 rounded text-[9px] font-black uppercase tracking-tighter shadow-sm;
}
.vip-tag.diamond { @apply bg-purple-100 text-purple-700 border border-purple-200; }
.vip-tag.gold { @apply bg-yellow-100 text-yellow-700 border border-yellow-200; }
.vip-tag.silver { @apply bg-blue-100 text-blue-700 border border-blue-200; }
.vip-tag.normal { @apply bg-gray-100 text-gray-500 border border-gray-200; }
.diamond { background: #e0e7ff; color: #4338ca; border: 1px solid #c7d2fe; }
.gold { background: #fef3c7; color: #b45309; border: 1px solid #fde68a; }
.silver { background: #f3f4f6; color: #4b5563; border: 1px solid #e5e7eb; }
.normal { background: #f9fafb; color: #9ca3af; }
</style>