<script setup>
import { computed, reactive, ref, watch } from 'vue'

const props = defineProps({
  currentUser: {
    type: Object,
    default: null,
  },
})

const emit = defineEmits(['login'])

const users = ref([
  { id: 1, name: 'Тестовый пользователь', email: 'test@test.test', password: '12345678', role: 'user' },
  { id: 2, name: 'Анна', email: 'admin@example.com', password: 'admin123', role: 'admin' },
])

const activeTab = ref('login')
const message = ref('')
const messageType = ref('info')

const loginForm = reactive({
  email: '',
  password: '',
})

const registerForm = reactive({
  name: '',
  email: '',
  password: '',
})

const canRegister = computed(() => props.currentUser?.role === 'admin')

watch(
  canRegister,
  (allowed) => {
    if (!allowed) {
      activeTab.value = 'login'
    }
  },
  { immediate: true },
)

const setMessage = (text, type = 'info') => {
  message.value = text
  messageType.value = type
}

const clearMessage = () => {
  message.value = ''
}

const onLogin = () => {
  clearMessage()

  const foundUser = users.value.find(
    (user) => user.email === loginForm.email.trim() && user.password === loginForm.password,
  )

  if (!foundUser) {
    setMessage('Неверный email или пароль.', 'error')
    return
  }

  emit('login', { ...foundUser })
  setMessage(`Вы вошли как ${foundUser.name}.`, 'success')
}

const onRegister = () => {
  clearMessage()

  if (!canRegister.value) {
    setMessage('Доступ к регистрации есть только у администратора.', 'error')
    return
  }

  const exists = users.value.some((user) => user.email === registerForm.email.trim())

  if (exists) {
    setMessage('Пользователь с таким email уже существует.', 'error')
    return
  }

  const newUser = {
    id: Date.now(),
    name: registerForm.name.trim(),
    email: registerForm.email.trim(),
    password: registerForm.password,
    role: 'user',
  }

  users.value.push(newUser)
  registerForm.name = ''
  registerForm.email = ''
  registerForm.password = ''
  setMessage('Новый пользователь успешно зарегистрирован.', 'success')
}
</script>

<template>
  <section class="auth-card">
    <h1>Авторизация</h1>

    <div v-if="canRegister" class="tabs">
      <button
        type="button"
        class="tab"
        :class="{ active: activeTab === 'register' }"
        @click="activeTab = activeTab === 'register' ? 'login' : 'register'"
      >
        {{ activeTab === 'register' ? 'К авторизации' : 'Регистрация' }}
      </button>
    </div>

    <form v-if="activeTab !== 'register'" class="form" @submit.prevent="onLogin">
      <label>
        Email
        <input v-model="loginForm.email" type="email" placeholder="test@test.test" required />
      </label>
      <label>
        Пароль
        <input v-model="loginForm.password" type="password" placeholder="Введите пароль" required />
      </label>
      <button type="submit" class="submit-btn">Войти</button>
    </form>

    <form v-else class="form" @submit.prevent="onRegister">
      <label>
        Имя
        <input v-model="registerForm.name" type="text" placeholder="Имя пользователя" required />
      </label>
      <label>
        Email
        <input v-model="registerForm.email" type="email" placeholder="new@example.com" required />
      </label>
      <label>
        Пароль
        <input
          v-model="registerForm.password"
          type="password"
          placeholder="Минимум 6 символов"
          minlength="6"
          required
        />
      </label>
      <button type="submit" class="submit-btn">Создать пользователя</button>
    </form>

    <p v-if="message" class="message" :class="messageType">{{ message }}</p>
  </section>
</template>

<style scoped>
.auth-card {
  width: 100%;
  max-width: 400px;
  box-sizing: border-box;
  margin: 0 auto;
  background: #ffffff;
  border: 1px solid #e2e8f0;
  box-shadow: 0 16px 44px rgba(15, 23, 42, 0.08);
  border-radius: 18px;
  padding: 30px;
}

h1 {
  margin: 0 0 18px;
  color: #0f172a;
  font-size: 1.7rem;
}

.tabs {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 18px;
}

.tab {
  border: none;
  border-radius: 10px;
  padding: 8px 14px;
  background: #e2e8f0;
  color: #334155;
  cursor: pointer;
  font-weight: 600;
}

.tab.active {
  background: #2563eb;
  color: #ffffff;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

label {
  display: flex;
  flex-direction: column;
  gap: 7px;
  color: #334155;
  font-size: 0.93rem;
}

input {
  border: 1px solid #cbd5e1;
  border-radius: 10px;
  padding: 11px 12px;
  font-size: 0.95rem;
  outline: none;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

input:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 4px rgba(37, 99, 235, 0.12);
}

.submit-btn {
  margin-top: 6px;
  border: none;
  border-radius: 10px;
  padding: 11px 14px;
  background: #2563eb;
  color: #ffffff;
  font-weight: 600;
  cursor: pointer;
}

.message {
  margin: 16px 0 0;
  padding: 10px 12px;
  border-radius: 10px;
  font-size: 0.9rem;
}

.message.info {
  background: #eff6ff;
  color: #1d4ed8;
}

.message.success {
  background: #ecfdf3;
  color: #166534;
}

.message.error {
  background: #fef2f2;
  color: #b91c1c;
}

@media (max-width: 768px) {
  .auth-card {
    max-width: 100%;
    padding: 22px 16px;
    border-radius: 14px;
  }

  h1 {
    font-size: 1.45rem;
  }

  label {
    font-size: 0.95rem;
  }

  input,
  .submit-btn {
    min-height: 44px;
    font-size: 1rem;
  }
}
</style>
