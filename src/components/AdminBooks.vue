<template>
  <div class="max-w-7xl mx-auto py-10 px-6 font-sans relative">

    <div class="absolute top-0 right-0 w-96 h-96 bg-indigo-50 rounded-full blur-[100px] -z-10"></div>

    <header class="mb-10 flex flex-col md:flex-row md:items-center justify-between gap-6">
      <div class="flex items-center space-x-4">
        <h1 class="text-3xl font-black text-gray-900 tracking-tight">Vault Inventory</h1>
        <span class="px-3 py-1 bg-gray-900 text-white text-[10px] font-bold rounded-lg uppercase tracking-widest">
          {{ books.length }} Items
        </span>
      </div>

      <div class="relative w-full md:w-96 group">
        <input v-model="inventorySearch" type="text" placeholder="Search by title or author..."
               class="w-full p-4 pl-12 bg-white/60 backdrop-blur-md border border-gray-200 rounded-2xl focus:ring-4 focus:ring-indigo-100/50 focus:border-indigo-400 transition-all font-bold text-gray-900 shadow-sm" />
        <svg class="absolute left-4 top-1/2 -translate-y-1/2 w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>
      </div>
    </header>

    <div class="grid grid-cols-12 gap-8 items-start">

      <aside class="col-span-12 lg:col-span-4">
        <div class="bg-white/70 backdrop-blur-xl border border-white rounded-[2rem] p-8 shadow-xl shadow-gray-200/50 sticky top-24">
          <h2 class="text-xl font-black text-gray-900 mb-8 flex items-center">
            <span class="w-2 h-6 bg-indigo-600 rounded-full mr-3"></span>
            Add New Asset
          </h2>

          <form @submit.prevent="handleCreate" class="space-y-6">
            <div class="space-y-2">
              <label class="text-xs font-black text-gray-400 uppercase tracking-widest ml-1">Title</label>
              <input v-model="createForm.book_name" type="text" placeholder="Book Name" required
                     class="w-full p-4 bg-gray-50 border-none rounded-xl focus:ring-2 focus:ring-indigo-500 font-bold text-gray-900" />
            </div>

            <div class="grid grid-cols-2 gap-4">
              <div class="space-y-2">
                <label class="text-xs font-black text-gray-400 uppercase tracking-widest ml-1">Price (¥)</label>
                <input v-model="createForm.price" type="number" step="0.01" required
                       class="w-full p-4 bg-gray-50 border-none rounded-xl focus:ring-2 focus:ring-indigo-500 font-bold text-gray-900" />
              </div>
              <div class="space-y-2">
                <label class="text-xs font-black text-gray-400 uppercase tracking-widest ml-1">Stock</label>
                <input v-model="createForm.stock" type="number" required
                       class="w-full p-4 bg-gray-50 border-none rounded-xl focus:ring-2 focus:ring-indigo-500 font-bold text-gray-900" />
              </div>
            </div>

            <div class="space-y-2">
              <label class="text-xs font-black text-gray-400 uppercase tracking-widest ml-1">Category</label>
              <select v-model="createForm.category_id" required
                      class="w-full p-4 bg-gray-50 border-none rounded-xl focus:ring-2 focus:ring-indigo-500 font-bold text-gray-700 appearance-none cursor-pointer">
                <option value="" disabled>Select Domain</option>
                <option v-for="cat in categories" :key="cat.category_id" :value="cat.category_id">{{ cat.category_name }}</option>
              </select>
            </div>

            <button type="submit" class="w-full py-4 bg-gray-900 text-white rounded-xl font-black text-sm uppercase tracking-widest hover:bg-indigo-600 transition-all active:scale-95 shadow-lg">
              Confirm Addition
            </button>
          </form>
        </div>
      </aside>

      <main class="col-span-12 lg:col-span-8 space-y-4">
        <div v-if="filteredInventory.length === 0" class="py-20 text-center bg-white/50 rounded-3xl border border-dashed border-gray-300">
          <p class="text-gray-400 font-bold">No results matching your query.</p>
        </div>

        <div v-for="book in filteredInventory" :key="book.book_id"
             @click="openDetail(book)"
             class="group flex items-center justify-between p-6 bg-white/60 backdrop-blur-md border border-white hover:border-indigo-100 rounded-3xl shadow-sm hover:shadow-xl hover:-translate-y-0.5 transition-all duration-300 cursor-pointer">

          <div class="flex items-center space-x-6">
            <div class="h-16 w-12 bg-gray-100 rounded-xl overflow-hidden shadow-sm flex-shrink-0">
              <img :src="`https://images.unsplash.com/photo-1544947950-fa07a98d237f?auto=format&fit=crop&q=80&w=150&sig=${book.book_id}`" class="object-cover w-full h-full" />
            </div>
            <div>
              <h3 class="text-lg font-black text-gray-900 group-hover:text-indigo-600 transition-colors">{{ book.book_name }}</h3>
              <p class="text-sm font-bold text-gray-400">{{ book.author }} · <span class="text-indigo-400">{{ book.category_name }}</span></p>
            </div>
          </div>

          <div class="flex items-center space-x-12">
            <div class="text-right">
              <p class="text-xl font-black text-gray-900">¥{{ book.price }}</p>
              <p class="text-[10px] font-bold text-gray-400 uppercase tracking-tighter">{{ book.stock }} in Stock</p>
            </div>
            <div class="w-10 h-10 flex items-center justify-center rounded-full bg-gray-50 text-gray-400 group-hover:bg-indigo-600 group-hover:text-white transition-all">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M9 5l7 7-7 7" /></svg>
            </div>
          </div>
        </div>
      </main>
    </div>

    <Transition name="modal">
      <div v-if="selectedBook" class="fixed inset-0 z-[100] flex items-center justify-center p-4 sm:p-6">
        <div class="absolute inset-0 bg-gray-900/40 backdrop-blur-md" @click="selectedBook = null"></div>

        <div class="relative bg-white rounded-[2.5rem] shadow-4xl w-full max-w-2xl overflow-hidden animate-zoomIn">
          <div class="p-10">
            <div class="flex flex-col md:flex-row gap-10">
              <div class="w-full md:w-2/5">
                <div class="aspect-[3/4] bg-gray-50 rounded-2xl shadow-inner overflow-hidden border border-gray-100">
                  <img :src="`https://images.unsplash.com/photo-1544947950-fa07a98d237f?auto=format&fit=crop&q=80&w=600&sig=${selectedBook.book_id}`" class="w-full h-full object-cover" />
                </div>
                <div class="mt-4 flex items-center justify-center space-x-2 text-gray-400">
                  <span class="text-[10px] font-black uppercase tracking-widest">Asset ID: {{ selectedBook.book_id }}</span>
                </div>
              </div>

              <div class="w-full md:w-3/5 space-y-6">
                <h3 class="text-2xl font-black text-gray-900">Modify Listing</h3>

                <div class="space-y-4">
                  <div class="space-y-1">
                    <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Book Title</label>
                    <input v-model="editForm.book_name" class="w-full p-4 bg-gray-50 rounded-xl font-bold text-gray-900 focus:ring-2 focus:ring-indigo-500 outline-none" />
                  </div>

                  <div class="grid grid-cols-2 gap-4">
                    <div class="space-y-1">
                      <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Price</label>
                      <input v-model="editForm.price" type="number" step="0.01" class="w-full p-4 bg-gray-50 rounded-xl font-bold text-gray-900 outline-none" />
                    </div>
                    <div class="space-y-1">
                      <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Stock</label>
                      <input v-model="editForm.stock" type="number" class="w-full p-4 bg-gray-50 rounded-xl font-bold text-gray-900 outline-none" />
                    </div>
                  </div>
                </div>

                <div class="pt-6 flex flex-col space-y-3">
                  <button @click="handleUpdate" class="w-full py-4 bg-indigo-600 text-white rounded-xl font-black text-sm uppercase tracking-widest shadow-lg hover:bg-indigo-700 transition-all">
                    Save Changes
                  </button>
                  <button @click="handleDelete" class="w-full py-4 bg-white text-red-500 border border-red-100 rounded-xl font-black text-sm uppercase tracking-widest hover:bg-red-50 transition-all">
                    Remove from Vault
                  </button>
                </div>
              </div>
            </div>
          </div>

          <button @click="selectedBook = null" class="absolute top-6 right-6 w-10 h-10 flex items-center justify-center bg-gray-50 rounded-full text-gray-400 hover:text-gray-900 transition-all">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M6 18L18 6M6 6l12 12" /></svg>
          </button>
        </div>
      </div>
    </Transition>

  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const books = ref([]);
const categories = ref([]);
const inventorySearch = ref('');
const selectedBook = ref(null);

const createForm = ref({ book_name: '', author: '', isbn: 'SYS-' + Date.now(), price: 0, stock: 0, category_id: '' });
const editForm = ref({ book_name: '', author: '', price: 0, stock: 0 });

const filteredInventory = computed(() => {
  if (!inventorySearch.value) return books.value;
  const q = inventorySearch.value.toLowerCase();
  return books.value.filter(b =>
      b.book_name.toLowerCase().includes(q) ||
      b.author.toLowerCase().includes(q)
  );
});

const openDetail = (book) => {
  selectedBook.value = book;
  editForm.value = { ...book };
};

const fetchCategories = async () => {
  const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/categories')).json();
  if (res.success) categories.value = res.data;
};

const fetchBooks = async () => {
  const res = await (await fetch('https://bookstore-backend-60vr.onrender.com/api/books?limit=200')).json();
  if (res.success) books.value = res.data;
};

const handleCreate = async () => {
  if (!createForm.value.category_id) return alert('Select Category');
  const res = await fetch('https://bookstore-backend-60vr.onrender.com/api/admin/books', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(createForm.value)
  });
  if ((await res.json()).success) { fetchBooks(); createForm.value = { book_name: '', author: '', isbn: 'SYS-' + Date.now(), price: 0, stock: 0, category_id: '' }; }
};

const handleUpdate = async () => {
  const res = await fetch(`https://bookstore-backend-60vr.onrender.com/${selectedBook.value.book_id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(editForm.value)
  });
  if ((await res.json()).success) { selectedBook.value = null; fetchBooks(); }
};

const handleDelete = async () => {
  if (!confirm('Permanently remove this book?')) return;
  const res = await fetch(`https://bookstore-backend-60vr.onrender.com/${selectedBook.value.book_id}`, { method: 'DELETE' });
  if ((await res.json()).success) { selectedBook.value = null; fetchBooks(); }
};

onMounted(() => { fetchCategories(); fetchBooks(); });
</script>

<style scoped>
.modal-enter-active, .modal-leave-active { transition: all 0.4s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; transform: scale(0.95); }

@keyframes zoomIn {
  from { opacity: 0; transform: scale(0.98) translateY(10px); }
  to { opacity: 1; transform: scale(1) translateY(0); }
}
.animate-zoomIn { animation: zoomIn 0.4s ease-out; }
</style>