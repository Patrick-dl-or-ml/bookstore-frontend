<template>
  <div class="py-12 sm:py-16 relative z-10 min-h-screen">
    <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8 relative z-20">

      <div class="bg-white/60 backdrop-blur-3xl border border-white/80 shadow-2xl rounded-[3rem] p-10 mb-12 flex flex-col md:flex-row md:items-center justify-between gap-8 relative overflow-hidden">
        <div class="absolute -top-24 -right-24 w-64 h-64 bg-indigo-100 rounded-full blur-3xl -z-10"></div>
        <div>
          <h1 class="text-5xl font-black tracking-tighter text-gray-900 mb-3">Curated Collection</h1>
          <p class="text-sm font-bold text-gray-500 uppercase tracking-widest">Discover handpicked premium reads for your mind.</p>
        </div>

        <div class="relative w-full md:w-[400px] group">
          <input v-model="searchQuery" @input="handleSearch" type="text" placeholder="Search titles, authors, or ISBN..."
                 class="block w-full px-8 py-5 bg-white/80 backdrop-blur-md border border-gray-100 rounded-[2rem] shadow-sm focus:ring-4 focus:ring-indigo-100 focus:border-indigo-400 font-bold text-gray-900 transition-all text-lg" />
          <svg class="absolute right-6 top-1/2 -translate-y-1/2 w-6 h-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>
        </div>
      </div>

      <div class="flex space-x-4 mb-12 overflow-x-auto pb-4 custom-scrollbar">
        <button v-for="cat in categories" :key="cat" @click="filterByCategory(cat)"
                :class="[activeCategory === cat ? 'bg-gray-900 text-white shadow-xl scale-105' : 'bg-white/60 text-gray-500 hover:bg-white hover:text-gray-900 border border-white']"
                class="px-8 py-4 text-xs font-black uppercase tracking-widest rounded-2xl transition-all whitespace-nowrap">
          {{ cat }}
        </button>
      </div>

      <div class="grid grid-cols-1 gap-y-12 gap-x-8 sm:grid-cols-2 lg:grid-cols-4 mb-16">
        <div v-if="books.length === 0" class="col-span-full py-32 text-center bg-white/40 rounded-[3rem] border border-white/60 shadow-sm">
          <p class="text-gray-400 text-lg font-black uppercase tracking-widest">No volumes found in this section.</p>
        </div>

        <div v-for="book in books" :key="book.book_id"
             @click="openBookDetail(book)"
             class="group relative flex flex-col bg-white/60 backdrop-blur-xl rounded-[2.5rem] p-5 shadow-lg border border-white/80 hover:shadow-2xl hover:-translate-y-2 transition-all duration-500 cursor-pointer">

          <div class="aspect-[3/4] w-full overflow-hidden rounded-[2rem] bg-gray-100 relative shadow-inner">
            <img :src="`https://images.unsplash.com/photo-1544947950-fa07a98d237f?auto=format&fit=crop&q=80&w=400&sig=${book.book_id}`" class="h-full w-full object-cover group-hover:scale-110 transition-transform duration-700" />

            <div v-if="book.stock > 0 && book.stock <= 5" class="absolute top-4 right-4 bg-red-500/90 backdrop-blur-sm text-white text-[10px] font-black px-3 py-1.5 rounded-xl shadow-lg uppercase tracking-widest">
              Only {{ book.stock }} Left
            </div>
            <div v-if="book.stock <= 0" class="absolute inset-0 bg-white/60 backdrop-blur-sm flex items-center justify-center">
              <span class="bg-gray-900 text-white px-6 py-2 rounded-2xl font-black text-xs uppercase tracking-widest">Out of Stock</span>
            </div>

            <div class="absolute inset-0 bg-gray-900/5 opacity-0 group-hover:opacity-100 transition-opacity flex items-end justify-center pb-6">
              <button @click.stop="addToCart(book)" :disabled="book.stock <= 0 || addingBookId === book.book_id" class="transform translate-y-4 group-hover:translate-y-0 transition-all bg-white/95 text-gray-900 px-8 py-4 rounded-2xl font-black text-xs uppercase tracking-widest shadow-2xl disabled:opacity-0 hover:bg-indigo-600 hover:text-white active:scale-95">
                <span v-if="addingBookId === book.book_id" class="text-emerald-500">✓ Added</span>
                <span v-else>Quick Add</span>
              </button>
            </div>
          </div>

          <div class="mt-6 flex flex-col flex-1 px-2 pb-2">
            <div class="flex justify-between items-start mb-2">
              <h3 class="text-xl font-black text-gray-900 line-clamp-1 flex-1 pr-4" :title="book.book_name">{{ book.book_name }}</h3>
              <span class="text-xl font-black text-indigo-600">¥{{ book.price }}</span>
            </div>
            <p class="text-xs text-gray-400 font-bold uppercase tracking-widest mb-4">{{ book.author }}</p>

            <div class="flex gap-2 mt-auto">
              <span class="bg-gray-100 text-gray-600 px-3 py-1 text-[9px] font-black uppercase tracking-widest rounded-lg">{{ book.quality || '全新' }}</span>
              <span class="bg-indigo-50 text-indigo-600 px-3 py-1 text-[9px] font-black uppercase tracking-widest rounded-lg">{{ book.category_name || 'Book' }}</span>
            </div>
          </div>
        </div>
      </div>

      <div v-if="totalPages > 1" class="flex flex-wrap items-center justify-center gap-2 mt-8 pb-8">
        <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1" class="w-12 h-12 flex items-center justify-center rounded-full bg-white/80 font-black shadow-sm disabled:opacity-40 transition-all">←</button>
        <template v-for="(item, index) in visiblePages" :key="index">
          <span v-if="item === '...'" class="px-2 text-gray-400 font-black tracking-widest self-end pb-2">...</span>
          <button v-else @click="changePage(item)" :class="[currentPage === item ? 'bg-gray-900 text-white scale-110 shadow-xl' : 'bg-white/80 text-gray-600 hover:bg-white']" class="w-12 h-12 rounded-full font-black text-sm flex items-center justify-center transition-all">{{ item }}</button>
        </template>
        <button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages" class="w-12 h-12 flex items-center justify-center rounded-full bg-white/80 font-black shadow-sm disabled:opacity-40 transition-all">→</button>
      </div>
    </div>

    <Transition name="modal">
      <div v-if="selectedBook" class="fixed inset-0 z-[100] flex items-center justify-center p-4 sm:p-6">
        <div class="absolute inset-0 bg-gray-900/60 backdrop-blur-xl" @click="selectedBook = null"></div>

        <div class="relative bg-white/95 backdrop-blur-3xl w-full max-w-5xl rounded-[3.5rem] shadow-4xl overflow-hidden animate-zoomIn flex flex-col md:flex-row max-h-[90vh]">

          <div class="w-full md:w-2/5 bg-gray-50 p-8 flex items-center justify-center relative">
            <div class="absolute inset-0 bg-gradient-to-br from-indigo-100/50 to-purple-50/50"></div>
            <div class="relative z-10 w-full aspect-[3/4] rounded-[2rem] shadow-2xl overflow-hidden ring-8 ring-white">
              <img :src="`https://images.unsplash.com/photo-1544947950-fa07a98d237f?auto=format&fit=crop&q=80&w=800&sig=${selectedBook.book_id}`" class="w-full h-full object-cover" />
            </div>

            <button @click="selectedBook = null" class="absolute top-6 left-6 w-12 h-12 flex items-center justify-center bg-white/80 backdrop-blur-md rounded-full text-gray-900 shadow-lg hover:scale-110 transition-all md:hidden">
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M15 19l-7-7 7-7" /></svg>
            </button>
          </div>

          <div class="w-full md:w-3/5 p-10 md:p-16 flex flex-col overflow-y-auto custom-scrollbar bg-white">

            <div class="flex gap-3 mb-6">
              <span class="px-4 py-1.5 bg-indigo-50 border border-indigo-100 text-indigo-600 text-[10px] font-black uppercase tracking-widest rounded-xl">{{ selectedBook.category_name }}</span>
              <span class="px-4 py-1.5 bg-gray-100 border border-gray-200 text-gray-600 text-[10px] font-black uppercase tracking-widest rounded-xl">{{ selectedBook.quality || '全新' }} Condition</span>
            </div>

            <h2 class="text-4xl font-black text-gray-900 leading-tight mb-2">{{ selectedBook.book_name }}</h2>
            <p class="text-lg font-bold text-gray-400 uppercase tracking-widest mb-8">By {{ selectedBook.author }}</p>

            <div class="flex items-end justify-between mb-8 pb-8 border-b border-gray-100">
              <div>
                <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">Current Price</p>
                <p class="text-5xl font-black text-gray-900 italic">¥{{ selectedBook.price }}</p>
              </div>
              <div class="text-right">
                <p class="text-[10px] font-black uppercase tracking-widest mb-1" :class="selectedBook.stock < 10 ? 'text-red-400' : 'text-emerald-500'">
                  {{ selectedBook.stock > 0 ? 'In Stock' : 'Out of Stock' }}
                </p>
                <p class="text-xl font-black text-gray-900">{{ selectedBook.stock }} Units</p>
              </div>
            </div>

            <button @click="addToCart(selectedBook)" :disabled="selectedBook.stock <= 0 || addingBookId === selectedBook.book_id"
                    class="w-full py-6 rounded-[2rem] font-black text-sm uppercase tracking-[0.3em] shadow-2xl transition-all active:scale-95 mb-12"
                    :class="selectedBook.stock <= 0 ? 'bg-gray-200 text-gray-400 cursor-not-allowed' : 'bg-gray-900 text-white hover:bg-indigo-600 hover:shadow-indigo-200/50'">
              <span v-if="addingBookId === selectedBook.book_id" class="text-emerald-400 flex items-center justify-center gap-2">
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7" /></svg>
                Added to Cart
              </span>
              <span v-else>{{ selectedBook.stock > 0 ? 'Add to Shopping Bag' : 'Currently Unavailable' }}</span>
            </button>

            <div class="grid grid-cols-2 gap-8 mb-10">
              <div>
                <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">Publisher</p>
                <p class="text-sm font-bold text-gray-900">{{ selectedBook.publisher || 'Independent / Unknown' }}</p>
              </div>
              <div>
                <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">ISBN</p>
                <p class="text-sm font-bold text-gray-900 font-mono">{{ selectedBook.isbn || 'N/A' }}</p>
              </div>
            </div>

            <div>
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-4">Synopsis / Description</p>
              <div class="prose prose-sm text-gray-600 leading-relaxed font-medium">
                {{ selectedBook.description || 'No detailed description provided by the publisher for this edition. Please check the metadata tags for more information regarding its category and condition.' }}
              </div>
            </div>

          </div>

          <button @click="selectedBook = null" class="absolute top-8 right-8 w-14 h-14 hidden md:flex items-center justify-center bg-gray-100/80 backdrop-blur-sm rounded-full text-gray-500 hover:bg-gray-900 hover:text-white transition-all">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M6 18L18 6M6 6l12 12" /></svg>
          </button>
        </div>
      </div>
    </Transition>

  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const books = ref([]);
const addingBookId = ref(null);
const categories = ref(['All']);
const searchQuery = ref('');
const activeCategory = ref('All');
let searchTimer = null;

const currentPage = ref(1);
const limit = ref(12); // 一页放 12 个商品卡片
const totalPages = ref(1);

// 🌟 新增：选中的书籍状态
const selectedBook = ref(null);

const visiblePages = computed(() => {
  const total = totalPages.value, current = currentPage.value, range = [];
  for (let i = Math.max(2, current - 1); i <= Math.min(total - 1, current + 1); i++) range.push(i);
  if (current - 1 > 2) range.unshift('...');
  if (current + 1 < total - 1) range.push('...');
  range.unshift(1);
  if (total > 1) range.push(total);
  return range;
});

const getCurrentUserId = () => JSON.parse(localStorage.getItem('user') || '{}').id || 1;

const fetchCategories = async () => {
  try {
    const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/categories')).json();
    if (res.success) categories.value = ['All', ...res.data.map(i => i.category_name)];
  } catch (e) { console.error(e); }
};

const fetchBooks = async () => {
  try {
    const url = new URL('https://bookstore-backend-60vr.onrender.com/api/books');
    url.searchParams.append('page', currentPage.value);
    url.searchParams.append('limit', limit.value);
    if (searchQuery.value) url.searchParams.append('keyword', searchQuery.value);
    if (activeCategory.value !== 'All') url.searchParams.append('category', activeCategory.value);

    const res = await (await fetch(url)).json();
    if (res.success) {
      books.value = res.data;
      totalPages.value = res.totalPages || 1;
    }
  } catch (e) { console.error(e); }
};

const changePage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page;
    fetchBooks();
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }
};

const handleSearch = () => {
  clearTimeout(searchTimer);
  searchTimer = setTimeout(() => { currentPage.value = 1; fetchBooks(); }, 300);
};

const filterByCategory = (cat) => {
  activeCategory.value = cat;
  currentPage.value = 1;
  fetchBooks();
};

// 🌟 打开详情弹窗
const openBookDetail = (book) => {
  selectedBook.value = book;
};

// 加购逻辑
const addToCart = async (book) => {
  addingBookId.value = book.book_id;
  try {
    await fetch('https://bookstore-backend-60vr.onrender.com/api/cart', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ consumer_id: getCurrentUserId(), book_id: book.book_id })
    });
    // 模拟 1.5 秒的加载动画
    setTimeout(() => { addingBookId.value = null; }, 1500);
  } catch (error) {
    console.error('Add to cart failed', error);
    addingBookId.value = null;
  }
};

onMounted(() => {
  fetchCategories();
  fetchBooks();
});
</script>

<style scoped>
.custom-scrollbar::-webkit-scrollbar { width: 4px; display: none;} /* 隐藏滚动条但允许滚动 */
.custom-scrollbar:hover::-webkit-scrollbar { display: block; }
.custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
.custom-scrollbar::-webkit-scrollbar-thumb { background: #E5E7EB; border-radius: 4px; }

/* 详情页弹窗动画 */
.modal-enter-active, .modal-leave-active { transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1); }
.modal-enter-from, .modal-leave-to { opacity: 0; backdrop-filter: blur(0); }

@keyframes zoomIn {
  from { opacity: 0; transform: scale(0.95) translateY(20px); filter: blur(5px); }
  to { opacity: 1; transform: scale(1) translateY(0); filter: blur(0); }
}
.animate-zoomIn { animation: zoomIn 0.5s cubic-bezier(0.16, 1, 0.3, 1) forwards; }
</style>