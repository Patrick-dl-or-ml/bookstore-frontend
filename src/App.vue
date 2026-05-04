<template>
  <div class="min-h-screen bg-[radial-gradient(ellipse_at_top_right,_var(--tw-gradient-stops))] from-indigo-50 via-slate-50 to-purple-50 font-sans">

    <nav v-if="isLoggedIn" class="bg-white/80 backdrop-blur-md sticky top-0 z-50 shadow-sm border-b border-gray-100">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between h-16">
          <div class="flex">
            <div class="flex-shrink-0 flex items-center font-black text-2xl tracking-tighter text-gray-900 cursor-pointer" @click="currentView = 'store'">
              MODERN<span class="text-indigo-600">BOOK.</span>
            </div>
            <div class="ml-12 flex space-x-8">
              <button @click="currentView = 'store'" :class="[currentView === 'store' ? 'border-indigo-600 text-gray-900' : 'border-transparent text-gray-500 hover:text-gray-900', 'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-bold transition-all']">Store</button>
              <button @click="currentView = 'cart'" :class="[currentView === 'cart' ? 'border-indigo-600 text-gray-900' : 'border-transparent text-gray-500 hover:text-gray-900', 'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-bold transition-all']">My Cart</button>

              <!-- 👇 新增：普通用户查看自己订单的入口 👇 -->
              <button @click="currentView = 'history'" :class="[currentView === 'history' ? 'border-indigo-600 text-gray-900' : 'border-transparent text-gray-500 hover:text-gray-900', 'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-bold transition-all']">My Orders</button>

              <button v-if="currentUser.role === 'admin'" @click="currentView = 'manage'"
                      :class="[currentView === 'manage' ? 'border-indigo-600 text-gray-900' : 'border-transparent text-gray-500 hover:text-gray-900', 'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-bold transition-all']">
                Inventory
              </button>

              <button v-if="currentUser.role === 'admin'" @click="currentView = 'users'"
                      :class="[currentView === 'users' ? 'border-indigo-600 text-gray-900' : 'border-transparent text-gray-500 hover:text-gray-900', 'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-bold transition-all']">
                Clients
              </button>

              <button v-if="currentUser.role === 'admin'" @click="currentView = 'orders'"
                      :class="[currentView === 'orders' ? 'border-indigo-600 text-gray-900' : 'border-transparent text-gray-500 hover:text-gray-900', 'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-bold transition-all']">
                Orders
              </button>

              <button v-if="currentUser.role === 'admin'" @click="currentView = 'admin'" :class="[currentView === 'admin' ? 'border-indigo-600 text-gray-900' : 'border-transparent text-gray-500 hover:text-gray-900', 'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-bold transition-all']">Dashboard</button>
            </div>
          </div>
          <div class="flex items-center space-x-4">
            <span class="text-sm font-bold text-gray-500">Hi, {{ currentUser.name }}</span>
            <button @click="handleLogout" class="text-xs font-bold text-red-400 hover:text-red-600 transition">Logout</button>
            <div @click="currentView = 'profile'" class="h-8 w-8 rounded-full bg-gray-900 text-white flex items-center justify-center font-bold text-xs cursor-pointer hover:ring-4 hover:ring-gray-100 transition-all">
              {{ currentUser.name?.charAt(0) }}
            </div>
          </div>
        </div>
      </div>
    </nav>

    <main>
      <transition name="fade" mode="out-in">
        <LoginView v-if="!isLoggedIn" @login-success="onLoginSuccess" />

        <div v-else>
          <BookStore v-if="currentView === 'store'" />
          <ShoppingCart v-else-if="currentView === 'cart'" />

          <!-- 👇 新增：渲染个人订单组件 👇 -->
          <MyOrders v-else-if="currentView === 'history'" />

          <UserProfile v-else-if="currentView === 'profile'" />
          <AdminDashboard v-else-if="currentView === 'admin' && currentUser.role === 'admin'" />
          <AdminBooks v-else-if="currentView === 'manage' && currentUser.role === 'admin'" />
          <AdminUsers v-else-if="currentView === 'users' && currentUser.role === 'admin'" />

          <AdminOrders v-else-if="currentView === 'orders' && currentUser.role === 'admin'" />
        </div>
      </transition>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import LoginView from './components/LoginView.vue'
import BookStore from './components/BookStore.vue'
import ShoppingCart from './components/ShoppingCart.vue'
import AdminDashboard from './components/AdminDashboard.vue'
import UserProfile from './components/UserProfile.vue'
import AdminBooks from './components/AdminBooks.vue'
import AdminUsers from './components/AdminUsers.vue'
import AdminOrders from './components/AdminOrders.vue'
// 👇 新增：引入 MyOrders 组件 👇
import MyOrders from './components/MyOrders.vue'

const isLoggedIn = ref(false)
const currentUser = ref({})
const currentView = ref('store')

onMounted(() => {
  const savedUser = localStorage.getItem('user')
  if (savedUser) {
    currentUser.value = JSON.parse(savedUser)
    isLoggedIn.value = true
  }
})

const onLoginSuccess = (user) => {
  currentUser.value = user
  isLoggedIn.value = true
  localStorage.setItem('user', JSON.stringify(user))
  currentView.value = 'store'
}

const handleLogout = () => {
  isLoggedIn.value = false
  currentUser.value = {}
  localStorage.removeItem('user')
  currentView.value = 'store'
}
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>