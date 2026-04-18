<template>
  <div class="min-h-[80vh] flex items-center justify-center px-4 font-sans">
    <div class="max-w-md w-full bg-white rounded-3xl shadow-2xl shadow-indigo-100 border border-gray-100 p-10 transition-all duration-500">

      <div class="text-center mb-10">
        <h2 class="text-3xl font-black text-gray-900">{{ isLoginMode ? 'Welcome Back' : 'Create Account' }}</h2>
        <p class="text-gray-500 mt-2 font-medium">
          {{ isLoginMode ? 'Please sign in to your account' : 'Join our bookstore community today' }}
        </p>
      </div>

      <form @submit.prevent="handleAuth" class="space-y-5">

        <div v-if="isLoginMode" class="flex bg-gray-100 p-1 rounded-2xl mb-6">
          <button type="button" @click="role = 'user'"
                  :class="[role === 'user' ? 'bg-white shadow-sm text-gray-900' : 'text-gray-500']"
                  class="flex-1 py-2 text-sm font-bold rounded-xl transition-all">Customer</button>
          <button type="button" @click="role = 'admin'"
                  :class="[role === 'admin' ? 'bg-white shadow-sm text-gray-900' : 'text-gray-500']"
                  class="flex-1 py-2 text-sm font-bold rounded-xl transition-all">Admin</button>
        </div>

        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">Username</label>
          <input v-model="form.username" type="text" required
                 class="w-full px-4 py-4 bg-gray-50 border-none rounded-2xl focus:ring-2 focus:ring-indigo-500 font-medium transition-all"
                 placeholder="Choose a username" />
        </div>

        <div v-if="!isLoginMode" class="space-y-5">
          <div>
            <label class="block text-sm font-bold text-gray-700 mb-2">Email</label>
            <input v-model="form.email" type="email" required
                   class="w-full px-4 py-4 bg-gray-50 border-none rounded-2xl focus:ring-2 focus:ring-indigo-500 font-medium transition-all"
                   placeholder="your@email.com" />
          </div>
          <div>
            <label class="block text-sm font-bold text-gray-700 mb-2">Phone</label>
            <input v-model="form.phone" type="tel" required
                   class="w-full px-4 py-4 bg-gray-50 border-none rounded-2xl focus:ring-2 focus:ring-indigo-500 font-medium transition-all"
                   placeholder="Your phone number" />
          </div>
        </div>

        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">Password</label>
          <input v-model="form.password" type="password" required
                 class="w-full px-4 py-4 bg-gray-50 border-none rounded-2xl focus:ring-2 focus:ring-indigo-500 font-medium transition-all"
                 placeholder="••••••••" />
        </div>

        <button type="submit" :disabled="loading"
                class="w-full py-4 mt-4 bg-indigo-600 text-white rounded-2xl font-bold hover:bg-indigo-700 transition-all shadow-lg shadow-indigo-100 disabled:bg-gray-400">
          {{ loading ? 'Processing...' : (isLoginMode ? 'Sign In' : 'Sign Up') }}
        </button>

      </form>

      <div class="mt-8 text-center">
        <p class="text-sm font-medium text-gray-500">
          {{ isLoginMode ? "Don't have an account?" : "Already have an account?" }}
          <button @click="toggleMode" class="font-bold text-indigo-600 hover:text-indigo-800 transition ml-1">
            {{ isLoginMode ? 'Sign up' : 'Sign in' }}
          </button>
        </p>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const emit = defineEmits(['login-success']);

const isLoginMode = ref(true); // 默认显示登录模式
const role = ref('user');
const loading = ref(false);

// 表单数据绑定
const form = ref({
  username: '',
  password: '',
  email: '',
  phone: ''
});

// 切换登录/注册模式时，清空表单
const toggleMode = () => {
  isLoginMode.value = !isLoginMode.value;
  form.value = { username: '', password: '', email: '', phone: '' };
};

// 核心逻辑：判断当前是登录还是注册，调不同的接口
const handleAuth = async () => {
  loading.value = true;
  const endpoint = isLoginMode.value ? '/api/login' : '/api/register';

  // 组装发给后端的数据
  const payload = { ...form.value };
  if (isLoginMode.value) {
    payload.role = role.value; // 如果是登录，带上角色参数
  }

  try {
    const res = await fetch(`https://bookstore-backend-60vr.onrender.com/${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    const result = await res.json();

    if (result.success) {
      // 不管是登录成功还是注册成功，都直接自动登录进入商城！
      emit('login-success', result.user);
      if (!isLoginMode.value) alert('🎉 注册成功！已自动为您登录。');
    } else {
      alert(result.message);
    }
  } catch (err) {
    alert('网络出错了，请检查后端是否启动');
  } finally {
    loading.value = false;
  }
};
</script>